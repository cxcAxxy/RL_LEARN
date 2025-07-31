# dof control 控制

创建完actor

env0 = gym.create_env(sim, env_lower, env_upper, 2)
cartpole0 = gym.create_actor(env0, cartpole_asset, initial_pose, 'cartpole', 0, 1)



Configure DOF properties

props = gym.get_actor_dof_properties(env0, cartpole0)
props["driveMode"] = (gymapi.DOF_MODE_POS, gymapi.DOF_MODE_POS)
props["stiffness"] = (5000.0, 5000.0)
props["damping"] = (100.0, 100.0)
gym.set_actor_dof_properties(env0, cartpole0, props)      #修改完要重新设置会去

通过gym.get_actor_dof_properties()，获取actor的所有的自由度dof，然后进行一个设置，理论上说之前创建的时候有一个默认的设置



 Set DOF drive targets

cart_dof_handle0 = gym.find_actor_dof_handle(env0, cartpole0, 'slider_to_cart')
pole_dof_handle0 = gym.find_actor_dof_handle(env0, cartpole0, 'cart_to_pole')
gym.set_dof_target_position(env0, cart_dof_handle0, 0)
gym.set_dof_target_position(env0, pole_dof_handle0, 0.25 * math.pi)

重点关照 find_actor_dof_handle，找到对应的actor的具体的dof属性，注意名字，按照gpt的说法应该是根据urdf的joint名字命名的，若是一个joint对应多个自由度会进行差分

还有一点就是gym.set_dof_target_position，根据dof的数值进行设置，不过注意的是应该可以一次设置很多个
gym.set_dof_target_velocity(env1, pole_dof_handle1, -2.0 * math.pi)
gym.apply_dof_effort(env3, pole_dof_handle3, 200) 

速度控制和力矩控制

pos = gym.get_dof_position(env2, cart_dof_handle2)，get_dof_position()

补充
gym.get_dof_position(env, dof_handle)    获取指定自由度的当前位置（单位：米或弧度）    float
gym.get_dof_velocity(env, dof_handle)    获取指定自由度的当前速度（单位：m/s 或 rad/s）    float
gym.get_dof_force(env, dof_handle)    获取施加在指定自由度上的总力/力矩（只读）    float
gym.get_dof_state_tensor(sim)    获取所有自由度的状态张量（位置和速度）    torch.Tensor
gym.get_dof_force_tensor(sim)    获取所有自由度的力/力矩张量    torch.Tensor
gym.get_actor_dof_states(env, actor_handle, state_type)  #返回特定环境的特定actor

注意gym.get_dof_state_tensor(sim),是得到整个环境的



#获取所有 DOF 状态张量

dof_states = gym.get_dof_state_tensor(sim)

传入 PyTorch tensor 接口

dof_states = gymtorch.wrap_tensor(dof_states)  # shape: (num_envs * num_dofs, 2)



#获取所有 position 和 velocity

dof_pos = dof_states[:, 0]
dof_vel = dof_states[:, 1]



 举个例子：第 i 个环境的第 j 个 DOF 的位置

i = 3  # 第几个环境
j = 1  # 环境中的第几个 DOF
pos_ij = dof_pos[i * num_dofs_per_env + j].item()

这个地方也来介绍一下对应的wrap_tensor
✅ 作用：
从 整个 simulation（sim）中所有环境（env）里的所有 DOF 中，获取位置和速度状态，并返回一个原始 GPU Tensor 的句柄（pointer）。
⚠️ 注意：
返回的不是 PyTorch Tensor，而是低级 GPU Tensor（类似裸数据），你不能直接拿来做数学计算或索引

总结来说，首先是设置dof属性，注意记得设置会去
属性就是有dof 的控制模式，还有"stiffness" "damping"等，这两个属性是在速度控制和位置控制设置有效，而且在速度模式下，stiffness不需要设置

最后补充一下源代码的一个点
sim_params.physx.num_position_iterations = 4
sim_params.physx.num_velocity_iterations = 1
