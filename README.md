rom collections import defaultdict, deque

class TaskScheduler:
    def __init__(self):
        self.graph = defaultdict(list) # adjacency list
        self.durations = {} # Duration of each task
        self.in_degree = defaultdict(int) # In-degree of each task
        self.tasks = set()

    def add_task(self, task, duration, dependencies=[]):
        self.durations[task] = duration
        self.tasks.add(task)
        for dep in dependencies:
            self.graph[dep].append(task)
            self.in_degree[task] += 1
        if task not in self.in_degree:
            self.in_degree[task] = 0

    def calculate_earliest_and_latest_times(self):
        # Step 1: Topological sort to determine the order of tasks
        topo_order = []
        queue = deque([task for task in self.tasks if self.in_degree[task] == 0])
        
        while queue:
            task = queue.popleft()
            topo_order.append(task)
            for neighbor in self.graph[task]:
                self.in_degree[neighbor] -= 1
                if self.in_degree[neighbor] == 0:
                    queue.append(neighbor)
        
        # Step 2: Calculate EST and EFT
        EST = {task: 0 for task in self.tasks}
        EFT = {task: 0 for task in self.tasks}
        
        for task in topo_order:
            EFT[task] = EST[task] + self.durations[task]
            for neighbor in self.graph[task]:
                EST[neighbor] = max(EST[neighbor], EFT[task])
        
        # Step 3: Calculate LFT and LST
        max_time = max(EFT.values())
        LFT = {task: max_time for task in self.tasks}
        LST = {task: 0 for task in self.tasks}
        
        for task in reversed(topo_order):
            for neighbor in self.graph[task]:
                LFT[task] = min(LFT[task], LST[neighbor])
            LST[task] = LFT[task] - self.durations[task]
        
        # Return results
        return max(EFT.values()), min(LFT.values())

# Example usage
scheduler = TaskScheduler()
scheduler.add_task('A', 3)
scheduler.add_task('B', 2, ['A'])
scheduler.add_task('C', 4, ['A'])
scheduler.add_task('D', 1, ['B', 'C'])

earliest_completion, latest_completion = scheduler.calculate_earliest_and_latest_times()
print(f"Earliest Completion Time: {earliest_completion}")
print(f"Latest Completion Time: {latest_completion}")
