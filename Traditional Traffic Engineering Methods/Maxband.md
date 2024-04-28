- [Little 等 - 1981 - MAXBAND A versatile program for setting signals o.pdf](https://github.com/SSW-Talon/Survey_RL_Light/files/15141078/Little.-.1981.-.MAXBAND.A.versatile.program.for.setting.signals.o.pdf)

![image](https://github.com/SSW-Talon/Survey_RL_Light/assets/70064164/8df29efb-01de-423b-96e2-ee537b9255c7)
![image](https://github.com/SSW-Talon/Survey_RL_Light/assets/70064164/2b34d7d1-0840-4cb7-8bdc-a101b1cc27c3)

变量说明：
    
- 所有变量中，带有\bar{\cdot}的标识下行方向（inbound），不带的标识上行方向（outbound）
- b为绿波带带宽，k为上下行带宽比例
- z为周期时长的倒数（便于模型求解）
- C_{max}, C_{min}为最大、最小周期时长（通常根据关键路口周期进行小幅调整）
- i为路口编号
- r_i为路口i的红灯时长（以周期为单位，如0.2个周期），w_i为红灯到绿波带距离（周期为单位），\tau_i为排队清空时间
- \delta_i为二元变量标识路口i左转相位lead or lag，l_i为左转时长
- d_i, t_i分别为路口i到i+1的距离、行程时间（周期为单位），e_i,f_i为绿波时速上下限，h_i,g_i为绿波时速变化量上下限

目标函数：最大化上下行带宽之和

约束：
- 约束1：上下行带宽大小约束
- 约束2：周期范围约束
- 约束3~4：带宽只能使用绿灯时间
- 约束5：带宽贯通性约束（包含了左转lead/lag优化）
- 约束6~7：绿波时速上下限
- 约束7~8：绿波时速变化量上下限

求解：混合整数线性规划，求解效率在百ms~s/子区

求解模型后，根据下式计算相位差：
\phi_i=0.5r_{i+1}+w_{i+1}+t_i-0.5r_i-w_i-\tau_i
