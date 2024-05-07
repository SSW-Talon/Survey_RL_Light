- [Little 等 - 1981 - MAXBAND A versatile program for setting signals o.pdf](https://github.com/SSW-Talon/Survey_RL_Light/files/15141078/Little.-.1981.-.MAXBAND.A.versatile.program.for.setting.signals.o.pdf)

<div>
    <img src="https://github.com/SSW-Talon/Survey_RL_Light/assets/70064164/8df29efb-01de-423b-96e2-ee537b9255c7" alt="image" style="float:left; width:55%;">
    <img src="https://github.com/SSW-Talon/Survey_RL_Light/assets/70064164/58f173e2-9471-450b-8120-d0b103d0b9fd" alt="image" style="float:right; width:40%;">
</div>

#### Key Constraint
![image](https://github.com/SSW-Talon/Survey_RL_Light/assets/70064164/01acdc03-0a78-4950-aa3a-30babd493330)

#### Key Constraint Inference
![image](https://github.com/SSW-Talon/Survey_RL_Light/assets/70064164/3d167417-4f86-4875-bd03-64c15b3f0896)

<div>
    <img src="https://github.com/SSW-Talon/Survey_RL_Light/assets/70064164/bf7efcfe-4844-4c27-b726-f523671f026b" alt="image" style="float:left; width:35%;">
    <img src="https://github.com/SSW-Talon/Survey_RL_Light/assets/70064164/ee8bd66c-51a8-488a-bae3-516da2ab40b5" alt="image" style="float:right; width:35%;">
    <img src="https://github.com/SSW-Talon/Survey_RL_Light/assets/70064164/870385e7-65be-4d55-9750-f3225c00436a" alt="image" style="float:right; width:29%;">
</div>



#### Advantages：
1. Relatively little input: network geometry, speeds/accelarate limitation, green splits.
2. Vislization and interpretable.

- Output: cycle length, offsets, suggested speeds, order of left turn phases.



```python
import pulp as pl
# Create Problem
prob = pl.LpProblem("Maxband_Optimization", pl.LpMaximize)

# Create Variable
b = pl.LpVariable("b", lowBound=0)
b_bar = pl.LpVariable("b_bar", lowBound=0)
z = pl.LpVariable("z", lowBound=0)
w = pl.LpVariable.dicts("w", range(num_intersection), lowBound=0)
w_bar = pl.LpVariable.dicts("w_bar", range(num_intersection), lowBound=0)
t = pl.LpVariable.dicts("t", range(num_intersection-1), lowBound=0)
t_bar = pl.LpVariable.dicts("t_bar", range(num_intersection-1), lowBound=0)
m = pl.LpVariable.dicts("m", range(num_intersection-1), lowBound=0, cat=pl.LpInteger)
delta = pl.LpVariable.dicts("delta", range(num_intersection), cat=pl.LpBinary)
delta_bar = pl.LpVariable.dicts("delta_bar", range(num_intersection), cat=pl.LpBinary)

# Objective Function
prob += b + k * b_bar, "Maximize band"

# Add Constraints
if k == 1:
    prob += b_bar == b, "Constraint on b_bar and b"
else:
    prob += (1-k)*b_bar >= (1-k)*k*b, "Constraint on b_bar and b"
prob += z >= 1/C_max, "Min cycle constraint"
prob += z <= 1/C_min, "Max cycle constraint"

for i in range(num_intersection):
    prob += w[i] + b <= 1 - r[i], f"Cyclical green constraint at intersection {i}"
    prob += w_bar[i] + b_bar <= 1 - r_bar[i], f"Cyclical green constraint at intersection {i} mirrored"

for i in range(num_intersection-1):
    prob += (w[i]+w_bar[i] - w[i+1]-w_bar[i+1] + t[i]+t_bar[i] 
             + delta[i]*l[i] - delta_bar[i]*l_bar[i] - delta[i+1]*l[i+1] + delta_bar[i+1]*l_bar[i+1] 
             - m[i] == (r[i+1] - r[i]) + (tau_bar[i]+tau[i+1]), f"Cycle balance between intersection {i}")

    prob += (d[i]/f[i])*z <= t[i], f"Speed minimum constraint between intersection {i}"
    prob += (d[i]/e[i])*z >= t[i], f"Speed maximum constraint between intersection {i}"
    prob += (d_bar[i]/f_bar[i])*z <= t_bar[i], f"Speed minimun constraint between intersection {i} mirrored"
    prob += (d_bar[i]/e_bar[i])*z >= t_bar[i], f"Speed maximun constraint between intersection {i} mirrored"

for i in range(num_intersection-2):
    prob += (d[i]/h[i])*z <= (d[i]/d[i+1])*t[i+1] - t[i], f"Deceleration constraint between intersection {i}"
    prob += (d[i]/g[i])*z >= (d[i]/d[i+1])*t[i+1] - t[i], f"Acceleration constraint between intersection {i}"
    prob += (d_bar[i]/h_bar[i])*z <= (d_bar[i]/d_bar[i+1])*t_bar[i+1] - t_bar[i], f"Deceleration constraint between intersection {i} mirrored"
    prob += (d_bar[i]/g_bar[i])*z >= (d_bar[i]/d_bar[i+1])*t_bar[i+1] - t_bar[i], f"Acceleration constraint between intersection {i} mirrored"

# Solve Problem
prob.solve()

# Print Result
print("\nPulp Optimization:")
print("Status:", pl.LpStatus[prob.status])
print("Objective Value:", pl.value(prob.objective) if pl.LpStatus[prob.status] != 'Infeasible' else "No solution")
print("Solution:")
print("cycle =", 1/z.varValue if pl.LpStatus[prob.status] != 'Infeasible' else "No solution")
