## 编程作业
为了添加`sys_trace`系统调用，需进行以下步骤：

### 1. **系统调用注册**
• 在系统调用表中注册`sys_trace`，编号为410，指向处理函数。

### 2. **任务控制块修改**
• 每个任务（`TaskControlBlock`）添加一个计数器数组或哈希表，记录各系统调用的调用次数。
```rust
struct TaskControlBlock {
    // 其他字段...
    syscall_counts: HashMap<usize, usize>, // 或 Vec<usize>
}
```

### 3. **系统调用分发器增强**
• 在调用具体处理函数前，增加对应系统调用号的计数。
```rust
fn handle_syscall(syscall_id: usize, ...) -> isize {
    let task = current_task();
    task.increment_syscall_count(syscall_id); // 增加计数
    match syscall_id {
        410 => sys_trace(...),
        // 其他调用处理
    }
}
```

### 4. **sys_trace实现**
处理三种请求类型：
• **读内存（0）**：将`id`视为指针，读取其地址的字节。
• **写内存（1）**：将`data`低8位写入`id`地址。
• **查询调用次数（2）**：将`id`对应的调用次数加1，返回新值。
```rust
fn sys_trace(req: usize, id: usize, data: usize) -> isize {
    match req {
        0 => read_byte(id),
        1 => write_byte(id, data),
        2 => {
            let task = current_task_mut();
            let count = task.get_syscall_count(id) + 1;
            task.set_syscall_count(id, count);
            count as isize
        }
        _ => -1,
    }
}
```

### 5. **任务调度支持**
• 任务切换时，确保`syscall_counts`随任务上下文保存和恢复。

### 关键知识点
1. **系统调用机制**：通过软中断和分发表实现系统调用路由。
2. **用户内存访问**：在无地址空间隔离时，直接访问用户指针需谨慎。
3. **状态管理**：每个任务独立维护系统调用次数，需数据结构支持高效查询。
4. **原子操作**：计数更新需确保正确性，单核下关闭中断可避免竞态。
5. **接口设计**：提供安全的方法（如`current_task_add`）封装内部状态。

### 总结
通过扩展任务结构和系统调用处理逻辑，实现了对系统调用历史的追踪。`sys_trace`的多功能设计体现了系统调用的灵活性，但也需注意用户指针的安全隐患。该实现为后续章节的进程隔离和更高级的调试功能奠定了基础。

## 简答

## 荣誉准则
在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

loukas

此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/6answer.html
https://github.com/LearningOS/os-rcore-classroom-2025s-rcore-rCore-Camp-Code-2025S/pull/6/files#diff-4fd8d8291f4e4a18241f0535bbcee20e7d4f7da14e95aeafec2ab9c11119c297

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。