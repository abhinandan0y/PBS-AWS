# PBS-AWS

#### how to integrate pbs portable batch system in linux . explain with an example

```
PBS (Portable Batch System) is a job scheduler used in high-performance computing (HPC) environments to manage and schedule jobs on a compute cluster. To integrate PBS on a Linux system, you'll need to install PBS, configure it, and then submit jobs to the PBS server.
```
Here’s a step-by-step guide on how to integrate and use PBS in Linux:
```
1. Install PBS Pro (Open Source PBS)
PBS Pro can be installed either by compiling from source or by using pre-built packages. Here’s how you can do it using packages (on Ubuntu, for example).
```
a. Add the PBS repository and install:
```bash
Copy code
sudo apt-get update
sudo apt-get install wget
wget https://github.com/openpbs/openpbs/releases/download/v20.0.1/openpbs_20.0.1.20201103_ubuntu20.04_amd64.deb
sudo dpkg -i openpbs_20.0.1.20201103_ubuntu20.04_amd64.deb
sudo apt-get -f install  # to fix any missing dependencies
```
b. Set up the PBS Pro Server and Client:
After installation, you need to initialize PBS, which will set up the PBS Server, Scheduler, and Mom.

```bash
Copy code
sudo /opt/pbs/libexec/pbs_postinstall
sudo systemctl enable pbs
sudo systemctl start pbs
You can verify that PBS is running using:
```
```bash
Copy code
sudo systemctl status pbs
```
2. Configure PBS Pro
Now, configure the PBS server to handle job submissions.

a. Add compute nodes:
By default, PBS adds the local machine as a compute node. To check or add additional compute nodes, you’ll modify the nodes file.

```bash
Copy code
sudo nano /var/spool/pbs/server_priv/nodes
```
Add your compute nodes here. For a single-node setup (local machine), it should look like:

```bash
Copy code
localhost np=4
```
This means the local machine has 4 processors available for PBS jobs.

b. Start PBS services on the nodes:
```bash
Copy code
sudo systemctl restart pbs
```
c. Check the status of the PBS environment:
```bash
Copy code
qmgr -c "p n"  # prints node configuration
pbsnodes -a    # checks the status of all nodes
```
3. Submit Jobs Using PBS
Once PBS is running, you can submit batch jobs.

a. Create a job script:
Create a simple PBS job script to submit. For example, create a file called my_job.pbs:

```bash
Copy code
#!/bin/bash
#PBS -N test_job          # Job name
#PBS -l select=1:ncpus=2  # Select resources (1 node, 2 CPUs)
#PBS -l walltime=00:10:00 # Max run time (hh:mm:ss)
#PBS -j oe                # Join output and error logs
#PBS -M user@example.com   # Email notifications (optional)
#PBS -m abe               # When to send email (abort, begin, end)
#PBS -o /home/user/job_output.log  # Output file location

# Start of job
cd $PBS_O_WORKDIR
echo "Running PBS job..."
sleep 60  # Simulate work with a sleep
echo "Job complete."
```
b. Submit the job to PBS:
Submit the job using the qsub command:

```bash
Copy code
qsub my_job.pbs
```
This will output a job ID (e.g., 12345.localhost), which you can use to track the job.

c. Check the status of jobs:
To check the status of submitted jobs:

```bash
Copy code
qstat  # Lists all running jobs
qstat -f <job_id>  # Shows detailed job information
```
d. Cancel a job:
If you need to cancel a job:

```bash
Copy code
qdel <job_id>
```
4. PBS Commands Summary
Here’s a summary of common PBS commands:

Submit a job: qsub <script>
Check job status: qstat
Cancel a job: qdel <job_id>
Node status: pbsnodes -a
Configure server: qmgr
PBS logs: Located in /var/spool/pbs/server_logs/
Example Workflow:
Write a PBS job script (my_job.pbs).
Submit it with qsub my_job.pbs.
Check the status with qstat.
If required, cancel the job with qdel <job_id>.
Conclusion
By installing and configuring PBS on your Linux system, you can manage batch jobs across a cluster or a single machine. PBS allows you to distribute computational tasks efficiently, making it a valuable tool in HPC environments.






#### what is the benefit if i can create my own shell script to run tasks on specific cpu and i can set  limits 

Creating your own shell script to run tasks on specific CPUs and setting resource limits can provide several key benefits, especially in multi-core or multi-CPU environments. Here's why this approach can be advantageous:

1. CPU Affinity (Pinning Tasks to Specific CPUs)
a. Improved Performance
When you pin tasks to specific CPUs using tools like taskset, you can ensure that a process consistently runs on the same CPU core(s). This can improve performance by reducing cache thrashing (the overhead of loading and reloading data into the CPU cache). As a result, the process can reuse data already in the cache, leading to better CPU efficiency and faster execution times.

b. Prevent Context Switching Overhead
By assigning tasks to specific CPUs, you minimize the overhead associated with context switching, where the operating system has to move tasks between cores. This can be beneficial for processes that are sensitive to latency or need real-time execution.

Example: Pinning a Process to a Specific CPU
```bash
Copy code
taskset -c 2 ./my_task.sh  # Run `my_task.sh` on CPU 2
```
2. Resource Isolation and Control
a. Limit CPU Usage
Using a custom shell script with tools like cgroups or ulimit, you can restrict how much CPU a task can use, ensuring that no single process monopolizes system resources. This is particularly useful in shared environments where you need to prevent tasks from overwhelming the CPU and affecting other running processes.

For example, using cgroups, you can limit a process to only use a specific percentage of CPU time.

b. Memory Limits
In addition to limiting CPU usage, you can set memory limits. This ensures that processes don’t consume excessive memory, leading to system slowdowns or out-of-memory (OOM) errors.

c. Disk I/O Control
Using ionice, you can set the priority of disk I/O for specific tasks. This ensures that disk-heavy tasks don’t degrade system performance by dominating I/O operations.

Example: Limit a Process's CPU and Memory Usage with cgroups
```bash
Copy code
cgcreate -g cpu,memory:/mytask
cgset -r cpu.shares=512 /mytask      # Assigns 50% CPU time
cgset -r memory.limit_in_bytes=1G /mytask  # Limits memory to 1 GB
cgexec -g cpu,memory:/mytask ./my_task.sh
```
3. Tailored Resource Allocation for Different Tasks
When you manage your own scripts, you have precise control over which tasks get which resources. This is especially useful in scenarios where you have:

Background tasks: that need to run with lower priority and minimal resource usage.
Time-sensitive tasks: that need higher priority and dedicated CPU/memory allocation.
Batch jobs or simulations: that can take advantage of multiple CPU cores or higher memory limits.
By writing custom scripts, you can design specific resource allocation strategies based on the nature of the tasks.

4. Improved System Stability
a. Avoid System Overload
By setting limits, you can prevent any single task from consuming excessive system resources, which could lead to system slowdowns, freezes, or crashes. This helps maintain system responsiveness and stability, even under heavy load.

b. Graceful Failure Handling
Custom scripts allow you to handle edge cases where a process consumes too many resources. For example, you can use timeouts or monitoring to automatically kill tasks that exceed certain resource thresholds, avoiding potential system failures.

5. Task Prioritization
With custom scripts, you can prioritize tasks using nice and renice, allowing you to control how aggressively tasks compete for CPU time. High-priority tasks (lower nice values) will be scheduled ahead of low-priority tasks.

Example: Lowering a Task’s Priority
```bash
Copy code
nice -n 10 ./background_task.sh  # Lower priority for background tasks
```
6. Automation and Scheduling Flexibility
Custom shell scripts allow you to automate the scheduling of tasks and fine-tune resource usage over time. You can create sophisticated scheduling workflows where tasks are executed based on specific conditions, dependencies, or resource availability.

Example: Running Tasks Periodically with Limits
Using cron, you can schedule your script to run at specific intervals:

```bash
Copy code
0 2 * * * taskset -c 1,2 ./my_task.sh  # Run task on CPUs 1 and 2 at 2 AM daily
```
7. Enhanced Control for Debugging and Profiling
When debugging performance issues, controlling which CPU cores and how much memory or CPU time a task can use can be valuable. For example, if you notice that a specific task is creating bottlenecks, you can tweak CPU pinning or resource limits in your script to investigate how different configurations affect performance.

8. Energy Efficiency
If you're working in a power-sensitive environment (like a data center), assigning tasks to specific CPUs and limiting resource usage can help save energy. By controlling how much CPU is used and when it’s used, you can reduce power consumption during low-demand periods.

Summary of Benefits:
Better CPU cache utilization (less cache thrashing).
Reduced context-switching overhead, leading to more consistent task performance.
Improved resource control: CPU, memory, disk I/O, and task priority management.
Increased system stability by preventing resource hogging.
Automation and flexibility through custom scheduling and limits.
Energy efficiency by controlling CPU usage patterns.
Creating your own shell scripts for managing CPU usage and setting resource limits is a powerful way to fine-tune system performance, especially in environments where multiple tasks or users share resources.






#### what is the benefit of usung pbs, if i can create my own shell script to run tasks on specific cpu and i can set  limits 

Using PBS (Portable Batch System) versus creating your own shell scripts to manage tasks and set resource limits provides different levels of automation, scalability, and efficiency. While shell scripts offer a certain level of control, PBS (or any job scheduler) brings a number of advantages when managing larger, more complex environments like clusters or high-performance computing (HPC) systems.

Here’s a comparison of the benefits PBS offers over using custom shell scripts:

1. Job Scheduling and Queue Management
```PBS:
Automated Job Scheduling: PBS can automatically queue jobs based on priority, available resources, or custom scheduling policies. It supports features like first-come, first-served (FCFS) scheduling, priority queues, and resource-based scheduling, ensuring jobs are run in an organized and efficient way.
Job Queues: PBS supports multiple job queues (e.g., high-priority, low-priority, or specific queues for different user groups). This ensures that resources are fairly allocated and that important jobs can bypass less critical tasks.
Shell Script:
Manual Scheduling: With a shell script, you would need to implement manual scheduling logic, such as running jobs sequentially, or rely on cron or at for time-based scheduling. However, it doesn’t have built-in queue management or prioritization features, making it cumbersome for handling multiple jobs with different priorities.
```
2. Scalability
```PBS:
Designed for Clusters: PBS is built for scalability in environments with many nodes and processors. It handles resource allocation across clusters with hundreds or thousands of CPUs. You can submit a job once, and PBS will automatically distribute it to available resources in the cluster.
Distributed Resource Management: PBS can manage jobs across different nodes, assigning the right number of CPUs, memory, and other resources per job, even in distributed systems.
Shell Script:
Limited to a Single Machine: Shell scripts typically operate on one machine, and if you want to scale to a multi-node environment, you would need to implement complex logic to distribute jobs across multiple machines. Managing distributed jobs manually using shell scripts is inefficient and error-prone.
```
3. Dynamic Resource Management
PBS:
Dynamic Allocation of Resources: PBS allows dynamic resource allocation, such as assigning more or fewer CPUs or memory to a job depending on current system load. This helps optimize resource usage in real time.
Preemption: PBS supports job preemption, meaning higher-priority jobs can interrupt lower-priority jobs when needed. The interrupted jobs can be resumed later, ensuring that important tasks get the required resources when they need them.
Shell Script:
Fixed Resources: While you can control CPU and memory limits using tools like taskset, cgroups, or ulimit, these limits are static and must be predefined. Shell scripts don’t have the ability to dynamically adjust resource usage based on current load or job priority.
No Preemption: Shell scripts don’t natively support job preemption. If a high-priority job needs to run, you would have to manually stop or reschedule other running jobs.
```
4. Error Handling and Job Resubmission
PBS:
Automatic Job Resubmission: PBS can automatically resubmit failed jobs or rerun jobs under specific conditions, such as after system crashes or hardware failures. This makes it easier to manage long-running jobs that may fail due to transient errors.
Advanced Error Reporting: PBS provides detailed logs and error tracking, which can help diagnose issues related to job execution, resource allocation, or node failures.
Shell Script:
Manual Error Handling: You would need to manually implement error handling and resubmission logic in your shell scripts. While this is possible, it requires significant effort and can become complex for handling edge cases (e.g., retries, dependencies, or partial failures).
Basic Logging: Shell scripts provide only basic error logging (through standard output or error), and additional logging mechanisms would need to be built manually.
```
5. Resource Optimization and Fair Share Scheduling
```PBS:
Resource Efficiency: PBS optimizes resource usage by packing jobs efficiently into available resources, ensuring that idle CPUs, memory, or other resources are minimized. This is done using algorithms that decide where and when jobs should run for maximum efficiency.
Fair Share Scheduling: PBS supports fair share scheduling, which ensures that different users or groups get a fair distribution of system resources over time. This is critical in environments where multiple users are sharing resources.
Shell Script:
Static Resource Allocation: Shell scripts allocate resources manually (with tools like taskset or cgroups), but they lack PBS's sophisticated scheduling algorithms. The system might be left with idle resources or over-allocated resources that are not optimized for overall system performance.
```
6. Job Dependencies and Workflow Automation
```PBS:
Job Dependencies: PBS supports job dependencies, where one job can start only after another job finishes successfully. This is critical in workflow automation, where jobs have interdependencies and need to be executed in a specific order.
Complex Workflows: PBS can manage complex workflows with multiple stages of processing, allowing different jobs to be run in sequence or parallel, depending on their dependencies.
Shell Script:
Limited Automation: You would need to manually code logic for job dependencies and sequence management in shell scripts, which could become cumbersome as workflows get more complex. Managing complex dependencies manually is error-prone and harder to scale.
```
7. User and Resource Accounting
```PBS:
Detailed Accounting: PBS has built-in accounting features that track how much CPU, memory, and other resources each job or user consumes. This is useful for system administrators who need to monitor resource usage or bill users for their share of the computing time.
Quotas and Limits: PBS can enforce quotas on resource usage, ensuring that no single user or group consumes more resources than allocated. It also supports limits on the number of jobs, CPUs, or memory that a user can use.
Shell Script:
Manual Tracking: With shell scripts, resource tracking and usage limits would have to be implemented manually. There’s no easy way to account for resource usage or enforce quotas across users in a multi-user environment.
```
8. Multi-User Support
```PBS:
Multi-User Environment: PBS is designed for multi-user environments where different users submit jobs to the same cluster. It provides mechanisms for user authentication, authorization, and resource allocation across multiple users.
User Isolation: PBS isolates user jobs and ensures that resource allocation is fair and in line with system policies, preventing resource contention between users.
Shell Script:
Manual User Management: Shell scripts don’t have built-in support for handling multi-user job submissions or user-level resource limits. You would need to manually manage user permissions, job submissions, and resource allocation for different users.
```
9. System-Wide Integration and Monitoring
```PBS:
System-Wide Resource Monitoring: PBS integrates with various monitoring tools to track system load, resource usage, and job status across all nodes in a cluster. This allows administrators to have a comprehensive view of system performance and job execution.
Shell Script:
Limited Monitoring: With shell scripts, you would need to rely on external monitoring tools like top, htop, or sar, and create custom monitoring scripts to track job performance across nodes or CPUs.
```
Conclusion:
When to Use PBS: PBS is ideal for large, multi-user environments (like HPC clusters) where you need automated job scheduling, resource management, dynamic scaling, and robust error handling. It simplifies managing large numbers of jobs and resources efficiently, making it the go-to solution for enterprise-level or scientific workloads.

When to Use Shell Scripts: For simpler, smaller-scale tasks on a single machine or in cases where you have complete control over the system and only need basic resource management, shell scripts may suffice. They offer more direct control but lack the sophisticated scheduling, scalability, and multi-user support that PBS provides.

If you’re managing a complex or multi-node environment, PBS (or similar job schedulers) is likely a more efficient and scalable solution compared to manually managing tasks with shell scripts.
