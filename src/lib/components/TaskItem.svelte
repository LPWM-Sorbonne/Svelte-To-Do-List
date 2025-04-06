 <script>
    // ⛑️ Auto-import pour éviter le bug récursif "TaskItem is not defined"
    import TaskItem from "./TaskItem.svelte";
  
    export let task;
    export let level = 0;
    export let selectedTaskIds;
    export let selectTask;
    export let toggleCheck;
    export let addSubtaskTo;
  </script>
  
  <div class="task" style="margin-left: 16px">
    <div class="task-row">
      <input
        type="checkbox"
        checked={selectedTaskIds.has(task.id)}
        on:change={() => toggleCheck(task)}
      />
      <span
        class:selected={selectedTaskIds.has(task.id)}
        class:completed={task.completed}
        on:click={() => selectTask(task.id)}
      >
        {task.text}
      </span>
      <button class="subtask-btn" on:click={() => addSubtaskTo(task)}>Sous-liste</button>
    </div>
  
    {#if task.children && task.children.length > 0}
      <div class="children-wrapper">
        {#each task.children as child (child.id)}
          <TaskItem
            task={child}
            level={level + 1}
            {selectedTaskIds}
            {selectTask}
            {toggleCheck}
            {addSubtaskTo}
          />
        {/each}
      </div>
    {/if}
  </div>
  
  <style>
    :global(body) {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
      background-color: var(--background-color, #f2f2f7);
      color: var(--text-color, #1c1c1e);
    }
  
    .task {
      margin-bottom: 4px;
    }
  
    .task-row {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      background: white;
      padding: 0.5rem 0.75rem;
      border-radius: 8px;
      transition: background-color 0.3s ease;
      border: 1px solid #e0e0e0;
      
      min-width: 250px;
    }
  
    .task-row:hover {
      background-color: #f0f0f5;
    }
  
    .task-row input[type="checkbox"] {
      transform: scale(1.1);
      cursor: pointer;
    }
  
    .task-row span {
      flex: 1;
      font-size: 15px;
      font-weight: 500;
      cursor: pointer;
      user-select: none;
    }
  
    .task-row span.selected {
      font-weight: 600;
      text-decoration: underline;
      color: #007aff;
    }
  
    .task-row span.completed {
      text-decoration: line-through;
      color: #999;
    }
  
    .subtask-btn {
      padding: 4px 10px;
      font-size: 13px;
      background: #e5e5ea;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      color: #000;
      transition: background 0.2s;
    }
  
    .subtask-btn:hover {
      background: #d1d1d6;
    }
  
    .children-wrapper {
      border-left: 2px solid #e5e5ea;
      margin-left: 8px;
      padding-left: 8px;
      margin-top: 4px;
    }
  </style>
  