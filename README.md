<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kanban Board - Drag & Drop</title>
    <style>
        /* --- Cài đặt cơ bản --- */
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background-color: #0079bf; /* Màu nền xanh đặc trưng của Trello */
            color: #172b4d;
            padding: 2rem;
            min-height: 100vh;
        }

        h1 {
            color: white;
            margin-bottom: 20px;
            text-align: center;
        }

        /* --- Bố cục bảng Kanban --- */
        .kanban-board {
            display: flex;
            align-items: flex-start;
            gap: 20px;
            overflow-x: auto; /* Cho phép cuộn ngang nếu nhiều cột */
            padding-bottom: 20px;
        }

        /* --- Thiết kế Cột (Column) --- */
        .kanban-column {
            background-color: #ebecf0;
            border-radius: 8px;
            width: 300px;
            min-width: 300px;
            padding: 15px;
            display: flex;
            flex-direction: column;
            max-height: 80vh;
        }

        .column-title {
            font-size: 1rem;
            font-weight: 700;
            margin-bottom: 15px;
            padding: 0 5px;
        }

        /* --- Khu vực chứa thẻ --- */
        .task-list {
            flex-grow: 1;
            min-height: 150px; /* Độ cao tối thiểu để dễ thả thẻ vào cột trống */
            display: flex;
            flex-direction: column;
            gap: 10px;
            overflow-y: auto; /* Cuộn dọc nếu cột quá nhiều thẻ */
            padding-bottom: 10px;
        }

        /* --- Thiết kế Thẻ công việc (Task Card) --- */
        .task {
            background-color: #ffffff;
            padding: 15px;
            border-radius: 6px;
            box-shadow: 0 1px 2px rgba(9, 30, 66, 0.25);
            cursor: grab; /* Biểu tượng bàn tay khi di chuột vào */
            font-size: 0.95rem;
            transition: box-shadow 0.2s, transform 0.2s;
        }

        .task:hover {
            background-color: #f4f5f7;
        }

        .task:active {
            cursor: grabbing; /* Đổi biểu tượng khi đang nắm giữ */
        }

        /* Hiệu ứng khi thẻ đang được kéo lên không trung */
        .task.dragging {
            opacity: 0.5;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }

        /* Hiệu ứng mờ nền cột khi kéo thẻ lướt qua */
        .task-list.drag-over {
            background-color: rgba(0, 0, 0, 0.05);
            border-radius: 4px;
        }
    </style>
</head>
<body>

    <h1>Kanban Board</h1>

    <div class="kanban-board">
        
        <!-- Cột Cần làm -->
        <div class="kanban-column">
            <h2 class="column-title">📝 Cần làm (To Do)</h2>
            <div class="task-list" id="todo">
                <!-- Thuộc tính draggable="true" cho phép trình duyệt biết thẻ này có thể kéo -->
                <div class="task" draggable="true">Lên ý tưởng thiết kế giao diện</div>
                <div class="task" draggable="true">Viết tài liệu hướng dẫn sử dụng API</div>
                <div class="task" draggable="true">Họp team chốt yêu cầu tuần tới</div>
            </div>
        </div>

        <!-- Cột Đang làm -->
        <div class="kanban-column">
            <h2 class="column-title">⏳ Đang làm (In Progress)</h2>
            <div class="task-list" id="in-progress">
                <div class="task" draggable="true">Lập trình chức năng Đăng nhập</div>
                <div class="task" draggable="true">Tối ưu hóa hình ảnh trên trang chủ</div>
            </div>
        </div>

        <!-- Cột Đã xong -->
        <div class="kanban-column">
            <h2 class="column-title">✅ Đã xong (Done)</h2>
            <div class="task-list" id="done">
                <div class="task" draggable="true">Cấu hình Server và Database</div>
            </div>
        </div>

    </div>

    <!-- Script xử lý sự kiện Kéo Thả -->
    <script>
        // Lấy tất cả các thẻ công việc và các cột chứa
        const tasks = document.querySelectorAll('.task');
        const taskLists = document.querySelectorAll('.task-list');

        let draggedTask = null; // Biến lưu trữ thẻ đang được kéo

        // 1. Cài đặt sự kiện cho từng thẻ (Task)
        tasks.forEach(task => {
            // Khi bắt đầu nhấc thẻ lên
            task.addEventListener('dragstart', () => {
                draggedTask = task;
                // Thêm class 'dragging' với độ trễ nhẹ để giữ nguyên hình dáng lúc kéo
                setTimeout(() => {
                    task.classList.add('dragging');
                }, 0);
            });

            // Khi buông thẻ ra
            task.addEventListener('dragend', () => {
                draggedTask = null;
                task.classList.remove('dragging');
            });
        });

        // 2. Cài đặt sự kiện cho các vùng chứa (Task List)
        taskLists.forEach(list => {
            // Khi kéo một thẻ lướt ngang qua vùng chứa
            list.addEventListener('dragover', (e) => {
                e.preventDefault(); // RẤT QUAN TRỌNG: Mặc định HTML5 không cho phép thả, phải chặn mặc định
                list.classList.add('drag-over'); // Thêm hiệu ứng màu nền để nhận biết
            });

            // Khi thẻ rời khỏi vùng chứa (chưa thả)
            list.addEventListener('dragleave', () => {
                list.classList.remove('drag-over');
            });

            // Khi người dùng THẢ thẻ xuống vùng chứa
            list.addEventListener('drop', (e) => {
                e.preventDefault();
                list.classList.remove('drag-over');
                
                // Di chuyển phần tử HTML của thẻ vào danh sách mới
                if (draggedTask) {
                    list.appendChild(draggedTask);
                }
            });
        });
    </script>

</body>
</html>
