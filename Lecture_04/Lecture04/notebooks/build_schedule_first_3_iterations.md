# build_schedule(): First 3 Iterations (Progressive Walkthrough)

This note explains the first **three** loop iterations of `build_schedule(individual)`.

We use the chromosome prefix:

```python
individual_prefix = [('J1',0), ('J2',0), ('J3',0)]
```

and the job definitions (machine, duration):

- J1: (M1,3) → (M2,2) → (M3,2)

- J2: (M2,2) → (M1,4) → (M3,3)

- J3: (M3,3) → (M2,3) → (M1,2)

- J4: (M1,2) → (M3,1) → (M2,4)

- J5: (M2,4) → (M3,3) → (M1,3)


## Progressive diagram (Iterations 1 → 2 → 3)

![Progressive diagram: first 3 iterations](iterations_1_2_3_progressive.png)

## Iteration 1

**Gene processed:** `(J1, 0)`  

**Operation:** `(M1, 3)`


### State *before* scheduling this gene

- `job_ready_time[J1] = 0`

- `machine_ready_time[M1] = 0`


### Compute start and end

- `start_time = max(0, 0) = 0`

- `end_time = start_time + duration = 0 + 3 = 3`


### Updates applied

- `job_ready_time[J1] ← 3`

- `machine_ready_time[M1] ← 3`


### State *after* this iteration

- Jobs: `J1=3, J2=0, J3=0, J4=0, J5=0`

- Machines: `M1=3, M2=0, M3=0`


![Iteration 1 diagram](iteration_1_diagram.png)

## Iteration 2

**Gene processed:** `(J2, 0)`  

**Operation:** `(M2, 2)`


### State *before* scheduling this gene

- `job_ready_time[J2] = 0`

- `machine_ready_time[M2] = 0`


### Compute start and end

- `start_time = max(0, 0) = 0`

- `end_time = start_time + duration = 0 + 2 = 2`


### Updates applied

- `job_ready_time[J2] ← 2`

- `machine_ready_time[M2] ← 2`


### State *after* this iteration

- Jobs: `J1=3, J2=2, J3=0, J4=0, J5=0`

- Machines: `M1=3, M2=2, M3=0`


![Iteration 2 diagram](iteration_2_diagram.png)

## Iteration 3

**Gene processed:** `(J3, 0)`  

**Operation:** `(M3, 3)`


### State *before* scheduling this gene

- `job_ready_time[J3] = 0`

- `machine_ready_time[M3] = 0`


### Compute start and end

- `start_time = max(0, 0) = 0`

- `end_time = start_time + duration = 0 + 3 = 3`


### Updates applied

- `job_ready_time[J3] ← 3`

- `machine_ready_time[M3] ← 3`


### State *after* this iteration

- Jobs: `J1=3, J2=2, J3=3, J4=0, J5=0`

- Machines: `M1=3, M2=2, M3=3`


![Iteration 3 diagram](iteration_3_diagram.png)

## How makespan is obtained at any moment

After building the schedule dictionary:

```python
return max(end for (_, end) in schedule.values())
```

This is the **finish time of the last operation** (the rightmost bar across all machines).
