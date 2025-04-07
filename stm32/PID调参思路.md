# PID 调参思路

## 调整顺序


 **(1) 初始设置**

- 将 **积分（I）** 和 **微分（D）** 增益设为 **零**，仅调整 **比例（P）** 增益，以防止过早引入不必要的复杂性。

 **(2) 调整比例增益（P）**
- 从 **较小的 P 值** 开始，逐步增大 **P**，直到系统输出出现**持续且等幅的明显振荡**。
- 此时的 P 值称为 **临界比例增益**，然后适当**降低 P**，使振荡不再明显，以确保系统具备一定的稳定性。

 **(3) 引入微分增益（D）**
- 在合适的 P 值基础上，逐步增加 **D**，以**抑制振荡**，提高系统的 **阻尼特性**，使其更趋向稳定。

 **(4) 引入积分增益（I）**
- 最后，缓慢增加 **I**，用于**消除稳态误差**，提高系统的精度。

---

## **针对不同情况的调参思路**


 ### **响应速度过慢**
- **增大 P**：提高系统对误差的响应强度
- **注意：** P 过大会导致系统振荡

### **系统出现振荡**
- **降低 P**：减少系统对误差的敏感度，避免过度纠正。
- **增加 D**：提供阻尼作用，抑制振荡，增强系统稳定性。
- **降低 I**：避免积分累积效应，减少长期误差对系统的影响。

### 系统出现高频抖动

- **降低 D**：减少对微小误差或噪声的放大。
- **降低 P**：减少对误差的过度响应。
- **适当滤波**

### **稳态误差过大**
- **增加 I**：增强积分作用，累积误差以消除稳态偏差，提高控制精度。
- **注意：** I 过大会导致系统响应变慢，并可能引发积分饱和或振荡。

---

## 积分饱和及其防范策略

### **积分饱和的概念**

在 PID 控制器的参数调整过程中，**积分饱和** 是一个需要特别关注的问题。当系统误差**持续存在**时，积分项会不断累积，可能导致控制器输出**超出其可控范围**，进而引发：
- **系统过冲**（超调严重）
- **响应迟缓**（系统恢复时间长）
- **控制不稳定**（振荡甚至发散）

### **应对策略**
针对积分饱和问题，可以采取以下策略：

- **积分分离：**  
  在误差较大时暂停积分作用，仅在误差较小时启用积分控制，以防止积分项过度累积。

   **示例代码：**

    ```c
    #define ERROR_THRESHOLD 100.0f  // 误差阈值，超过此值禁用积分项
  
    float PID_Control(float setpoint, float actual, float Kp, float Ki, float Kd) {
        static float integral = 0.0f;
        float error = setpoint - actual;
        float derivative;
        static float last_error = 0.0f;
  
        // 积分分离：只有误差较小时才计算积分
        if (fabs(error) < ERROR_THRESHOLD) {
            integral += Ki * error;
        }
  
        // 计算微分项
        derivative = Kd * (error - last_error);
  
        // 计算控制量
        float output = Kp * error + integral + derivative;
  
        // 更新上次误差
        last_error = error;
  
        return output;
    }
    ```

- **积分限幅：**  
  对积分项设置 **上下限**，限制其累积范围，避免积分项过大导致的饱和现象。

	**示例代码：**

    ```c
    #define INTEGRAL_MAX  1000.0f   // 积分项上限
    #define INTEGRAL_MIN -1000.0f   // 积分项下限
    
    float PID_Control(float setpoint, float actual, float Kp, float Ki, float Kd) {
        static float integral = 0.0f;
        float error = setpoint - actual;
        float derivative;
        static float last_error = 0.0f;
    
        // 计算积分项
        integral += Ki * error;
    
        // 积分限幅
        if (integral > INTEGRAL_MAX) integral = INTEGRAL_MAX;
        if (integral < INTEGRAL_MIN) integral = INTEGRAL_MIN;
    
        // 计算微分项
        derivative = Kd * (error - last_error);
    
        // 计算控制量
        float output = Kp * error + integral + derivative;
    
        // 更新上次误差
        last_error = error;
    
        return output;
    }
    
    ```



- **抗饱和积分（反向积分）：**  
  当控制器输出达到饱和状态时，暂停积分项的累积，待输出恢复正常后再继续积分。

​	**示例代码：**

```c
#define OUTPUT_MAX  1000.0f   // 控制器输出上限
#define OUTPUT_MIN -1000.0f   // 控制器输出下限

float PID_Control(float setpoint, float actual, float Kp, float Ki, float Kd) {
    static float integral = 0.0f;
    float error = setpoint - actual;
    float derivative;
    static float last_error = 0.0f;

    // 计算微分项
    derivative = Kd * (error - last_error);

    // 计算暂定输出值
    float temp_output = Kp * error + integral + derivative;

    // 抗饱和：如果输出超限，则暂停积分项
    if (temp_output < OUTPUT_MAX && temp_output > OUTPUT_MIN) {
        integral += Ki * error;
    }

    // 计算最终输出
    float output = Kp * error + integral + derivative;

    // 限制最终输出范围
    if (output > OUTPUT_MAX) output = OUTPUT_MAX;
    if (output < OUTPUT_MIN) output = OUTPUT_MIN;

    // 更新上次误差
    last_error = error;

    return output;
}

```



