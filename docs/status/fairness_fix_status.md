# Fairness Fix Status

最后更新时间：2026-04-01

## 结论

- 已修复 DeePC 仿真路径里“每个 control step 额外再采样一次 measurement”的问题。
- 现在在标准仿真入口里，`uniform DeePC`、`measurement-aware DeePC`、`xyz-only DeePC` 都复用 `Simulation` 在该步已经采到的同一条测量。
- `LQR` / `linear MPC` 仍然是状态反馈基线，使用 `x_current`，不消费同一条 corruption path，所以不能当作 measurement-aware 主张的同构公平对照。
- `xyz-only DeePC` 仍然不是强同构 baseline，只能作为 pruning / negative control。

## 改动文件

- [/home/roxy/deepc-paper/deepc/Simulator/simulation.py](/home/roxy/deepc-paper/deepc/Simulator/simulation.py)
- [/home/roxy/deepc-paper/deepc/Controllers/deepc.py](/home/roxy/deepc-paper/deepc/Controllers/deepc.py)
- [/home/roxy/deepc-paper/deepc/Controllers/lqr_tracking.py](/home/roxy/deepc-paper/deepc/Controllers/lqr_tracking.py)
- [/home/roxy/deepc-paper/deepc/Controllers/linear_mpc.py](/home/roxy/deepc-paper/deepc/Controllers/linear_mpc.py)

## 修复前数据流

- `Simulation.simulate()` 在每个采样点先调用 `system.measure_output(x_t)`，把结果记入 `y_data`。
- 然后 `Simulation` 把 `x_t` 直接传给 controller。
- `DeePC.compute_input(x_t)` 内部再次调用 `system.measure_output(x_t)`，导致同一 control step 里出现第二次 measurement sampling。
- 这意味着 DeePC 的控制输入和仿真结果记录的输出并不是同一条测量样本。
- `LQR` / `linear MPC` 始终直接使用 `x_t`，没有共享 DeePC 的 corruption 路径。

## 修复后数据流

- `Simulation.simulate()` 仍然在每个采样点先调用一次 `system.measure_output(x_t)`。
- 同一条 `y_t` 会同时用于：
  - 写入 `y_data`
  - 作为 `controller.compute_input(x_t, y_t)` 的输入
- `Simulation` 还把无噪声的 `y_true = C x_t` 一并写入结果，方便后续审计真实输出与 corrupted measurement 的差异。
- `DeePC.compute_input(x_t, y_t)` 如果已经收到 `y_t`，就不再调用 `measure_output`。
- `DeePC` 的 residual / slack 统计现在基于这条唯一的测量样本。
- `LQR` / `linear MPC` 的接口为了兼容 `Simulation`，接受了可选的 `y_current` 参数，但仍然忽略它并继续使用 `x_t`。

## Controller Audit

| Controller | Observation source | Corruption source | Controller 实际使用输入 | Fair for main claim? | Note |
| --- | --- | --- | --- | --- | --- |
| `uniform regularized DeePC` | `Simulation` 采到的 `y_t` | `quadcopter.measure_output()` 注入的 noise / bias / drift | `y_t` + past `y/u` history | yes | 现在不再额外重采样。 |
| `measurement_noise DeePC` | 同上 | 同上 | `y_t` + weighted slack / consistency | yes | 与 `uniform` 共享同一条测量样本。 |
| `residual-statistics DeePC` 变体 | 同上 | 同上 | `y_t` + residual-derived weights | yes | 只改 regularization，不改观测来源。 |
| `xyz-only DeePC` | 同上，但输出集合缩成 `xyz` | 同上 | `xyz` measured outputs + history | yes, but only as pruning control | 还伴随不同初始化/数据长度设置，不能伪装成强同构 baseline。 |
| `LQR` | `x_t` | none shared with DeePC | `x_t` | no | 只能作为 nominal engineering baseline。 |
| `linear MPC` | `x_t` | none shared with DeePC | `x_t` | no | 同上。 |

## 现在可用于方法主实验的 baseline

- `uniform regularized DeePC`
- `measurement_noise DeePC`
- residual-statistics DeePC 候选
- `xyz-only DeePC`，但只能当 pruning / negative control

## 仍然只能辅助参考的 baseline

- `LQR`
- `linear MPC`

## 仍然存在的公平性限制

- `LQR` / `linear MPC` 仍然不是同构 corruption-path baseline，它们不消费被污染的 measurement。
- `xyz-only DeePC` 不是纯粹的“只删输出”，它当前还带有不同的初始 excitation 和更长的数据长度设置，所以只能用于解释 pruning 效应，不能拿来冒充完全公平的强 baseline。
- `DeePC.compute_input(x_t, y_t)` 在脱离 `Simulation` 直接调用时，若 `y_t` 没传入，仍会回退到 on-demand measurement。这是兼容性保留，不是主实验路径。
- 本轮没有引入 observer / state estimator 来把 `LQR` / `MPC` 强行改造成 measurement-corrupted 对照，因为那会把 scope 扩到新的控制结构。

## 最小验证

验证命令：

```bash
MPLBACKEND=Agg /home/roxy/miniconda3/bin/conda run --no-capture-output -n delivery python - <<'PY'
import numpy as np
from Simulator.simulation import Simulation
from Controllers.deepc import DeePC
from Controllers.lqr import LQR
from quadcopter import Quadcopter
from trajectory_generator import TrajectoryGenerator

class DummyController:
    def __init__(self):
        self.calls = []
    def compute_input(self, x_current, y_current=None):
        self.calls.append(None if y_current is None else np.asarray(y_current).copy())
        return np.full(4, 0.5)

system = Quadcopter(
    h=0.1,
    measurement_config={"noise_std": 0.0, "yaw_bias": 0.0, "yaw_drift_per_sec": 0.0, "seed": 0},
    output_set="xyzpsi",
)
dummy = DummyController()
sim = Simulation(system, dummy, dt=0.001, t_final=0.3, verbose=False)
result = sim.simulate()
assert system._measurement_step == result["y"].shape[1]
assert len(dummy.calls) == result["y"].shape[1]
assert np.allclose(dummy.calls[0], result["y"][:, 0])

trajectory = TrajectoryGenerator(sort="step", system=system, duration=0.3, has_initial_ref=False)
deepc = DeePC(
    system,
    trajectory,
    initial_controller=LQR(system, noise=0.0, seed=0),
    is_regularized=True,
    prediction_horizon=2,
    t_ini=2,
    lambda_y=1.0,
    lambda_g=1.0,
    solver="CLARABEL",
)
original_measure_output = system.measure_output
def fail_measure_output(x):
    raise RuntimeError("unexpected extra measurement sampling")
system.measure_output = fail_measure_output
u = deepc.compute_input(system.x0, y_current=np.zeros(system.p))
assert u.shape == (system.m,)
system.measure_output = original_measure_output
print(f"simulation_measurements={int(system._measurement_step)}")
print(f"dummy_controller_calls={len(dummy.calls)}")
print(f"deepc_input_shape={list(u.shape)}")
print("deepc_extra_sampling=not triggered")
PY
```

验证结果：

- `simulation_measurements=3`
- `dummy_controller_calls=3`
- `deepc_input_shape=[4]`
- `deepc_extra_sampling=not triggered`

## 影响判断

- 当前 controller-to-controller 的主实验比较，在 observation / corruption 逻辑上已经比修复前更可解释、可复现。
- 但它仍不是“所有 controller 完全同构”的比较，因为 `LQR` / `linear MPC` 不是同一路径，`xyz-only DeePC` 也保留了结构性差异。
