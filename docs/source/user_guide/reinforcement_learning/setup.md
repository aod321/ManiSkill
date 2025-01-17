# Setup

ManiSkill environments are created by gymnasium's `make` function. The result is by default a "batched" environment where every input and output is batched. Note that this is not standard gymnasium API. If you want the standard gymnasium environemnt / vectorized environment API see the next sections.

```python
import mani_skill.envs
import gymnasium as gym
N = 4
env = gym.make("PickCube-v1", num_envs=N)
env.action_space # shape (N, D)
env.observation_space # shape (N, ...)
env.reset()
obs, rew, terminated, truncated, info = env.step(env.action_space.sample())
# obs (N, ...), rew (N, ), terminated (N, ), truncated (N, )
```

## Gym Environment API

If you want to use the CPU simulator / a single environment, you can apply the `CPUGymWrapper` which essentially unbatches everything and turns everything into numpy so the environment behaves just like a normal gym environment. The API for a gym environment is detailed on [their documentation](https://gymnasium.farama.org/api/env/).

```python
import mani_skill.envs
import gymnasium as gym
from mani_skill.utils.wrappers.gymnasium import CPUGymWrapper
N = 1
env = gym.make("PickCube-v1", num_envs=N)
env = CPUGymWrapper(env)
env.action_space # shape (D, )
env.observation_space # shape (...)
env.reset()
obs, rew, terminated, truncated, info = env.step(env.action_space.sample())
# obs (...), rew (float), terminated (bool), truncated (bool)
```

## Gym Vectorized Environment API

We adopt the gymnasium `VectorEnv` (also known as `AsyncVectorEnv`) interface as well and you can achieve that via a single wrapper so that your algorithms that assume `VectorEnv` interface can work seamlessly. The API for a vectorized gym environment is detailed on [their documentation](https://gymnasium.farama.org/api/vector/)

```python
import mani_skill.envs
import gymnasium as gym
from mani_skill.vector.wrappers.gymnasium import ManiSkillVectorEnv
N = 4
env = gym.make("PickCube-v1", num_envs=N)
env = ManiSkillVectorEnv(env, auto_reset=True, ignore_terminations=False)
env.action_space # shape (N, D)
env.single_action_space # shape (D, )
env.observation_space # shape (N, ...)
env.single_observation_space # shape (...)
env.reset()
obs, rew, terminated, truncated, info = env.step(env.action_space.sample())
# obs (N, ...), rew (N, ), terminated (N, ), truncated (N, )
```

You may also notice that there are two additional options when creating a vector env. The `auto_reset` argument controls whether to automatically reset a parallel environment when it is terminated or truncated. This is useful depending on algorithm. The `ignore_terminations` argument controls whether environments reset upon terminated being True. Like gymnasium vector environments, partial resets can occur where some parallel environments reset while others do not.

Note that for efficiency, everything returned by the environment will be a batched torch tensor on the GPU and not a batched numpy array on the CPU. This the only difference you may need to account for between ManiSkill vectorized environments and gymnasium vectorized environments.

