<!DOCTYPE html>
<html lang="th" class="h-full bg-slate-50">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบบริหารจัดการงานและติดตามภาระงาน (Workload & Task Tracker)</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Chart.js for Workload Charts -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Google Fonts: Noto Sans Thai -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+Thai:wght@300;400;500;600;700&display=swap" rel="stylesheet">

    <style>
        body {
            font-family: 'Noto Sans Thai', sans-serif;
        }
        /* Custom scrollbar for smooth Kanban scrolling */
        .kanban-col::-webkit-scrollbar {
            width: 6px;
        }
        .kanban-col::-webkit-scrollbar-thumb {
            background-color: #cbd5e1;
            border-radius: 9999px;
        }
    </style>
</head>
<body class="h-full text-slate-800 flex flex-col antialiased">

    <!-- Top Navigation Bar -->
    <header class="bg-indigo-900 text-white shadow-md sticky top-0 z-30">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
            <div class="flex items-center gap-3">
                <div class="bg-indigo-600 p-2 rounded-lg flex items-center justify-center shadow">
                    <i data-lucide="layout-dashboard" class="w-6 h-6 text-white"></i>
                </div>
                <div>
                    <h1 class="font-bold text-lg leading-tight">Workload & Task Manager</h1>
                    <p class="text-xs text-indigo-200">ระบบติดตามงาน & วิเคราะห์ภาระงานล้นมือ</p>
                </div>
            </div>
            
            <div class="flex items-center gap-3">
                <button onclick="openModal('projectModal')" class="bg-indigo-700 hover:bg-indigo-600 text-white text-sm font-medium px-3.5 py-2 rounded-lg flex items-center gap-1.5 transition shadow-sm">
                    <i data-lucide="folder-plus" class="w-4 h-4"></i>
                    <span class="hidden sm:inline">สร้างโปรเจกต์</span>
                </button>
                <button onclick="openModal('taskModal')" class="bg-amber-500 hover:bg-amber-400 text-slate-900 font-semibold text-sm px-4 py-2 rounded-lg flex items-center gap-1.5 transition shadow">
                    <i data-lucide="plus-circle" class="w-4 h-4"></i>
                    <span>เพิ่มงานใหม่</span>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Container -->
    <div class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 flex flex-col gap-6">

        <!-- Navigation Tabs & Capacity Setting -->
        <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 border-b border-slate-200 pb-4">
            <!-- View Tabs -->
            <nav class="flex space-x-2 bg-slate-200/70 p-1 rounded-xl w-full md:w-auto">
                <button id="tab-dashboard" onclick="switchTab('dashboard')" class="tab-btn flex-1 md:flex-initial px-4 py-2 text-sm font-medium rounded-lg flex items-center justify-center gap-2 transition bg-white text-indigo-900 shadow-sm">
                    <i data-lucide="pie-chart" class="w-4 h-4"></i> ภาพรวม & ภาระงาน
                </button>
                <button id="tab-kanban" onclick="switchTab('kanban')" class="tab-btn flex-1 md:flex-initial px-4 py-2 text-sm font-medium rounded-lg flex items-center justify-center gap-2 transition text-slate-600 hover:text-slate-900">
                    <i data-lucide="kanban" class="w-4 h-4"></i> บอร์ด Kanban
                </button>
                <button id="tab-followup" onclick="switchTab('followup')" class="tab-btn flex-1 md:flex-initial px-4 py-2 text-sm font-medium rounded-lg flex items-center justify-center gap-2 transition text-slate-600 hover:text-slate-900 relative">
                    <i data-lucide="bell-ring" class="w-4 h-4"></i> งานที่ต้องติดตาม
                    <span id="followupBadge" class="hidden bg-amber-500 text-white text-xs px-1.5 py-0.5 rounded-full font-bold">0</span>
                </button>
                <button id="tab-alltasks" onclick="switchTab('alltasks')" class="tab-btn flex-1 md:flex-initial px-4 py-2 text-sm font-medium rounded-lg flex items-center justify-center gap-2 transition text-slate-600 hover:text-slate-900">
                    <i data-lucide="list-todo" class="w-4 h-4"></i> รายการงานทั้งหมด
                </button>
            </nav>

            <!-- Capacity Setting Controls -->
            <div class="bg-amber-50 border border-amber-200 rounded-xl px-4 py-2 flex items-center gap-3 text-sm">
                <i data-lucide="zap" class="w-5 h-5 text-amber-600 shrink-0"></i>
                <div class="flex items-center gap-2">
                    <span class="text-xs text-amber-900 font-medium">ขีดจำกัดงาน/วัน:</span>
                    <input type="number" id="dailyCapacityInput" value="8" min="1" max="24" onchange="updateCapacity(this.value)" class="w-14 px-2 py-1 border border-amber-300 rounded text-center text-xs font-bold text-amber-900 bg-white focus:outline-none focus:ring-2 focus:ring-amber-500">
                    <span class="text-xs text-amber-800">ชม.</span>
                </div>
            </div>
        </div>

        <!-- Overload Alert Banner (Conditional) -->
        <div id="overloadBanner" class="hidden bg-rose-500 text-white p-4 rounded-xl shadow-lg flex items-center justify-between animate-pulse">
            <div class="flex items-center gap-3">
                <div class="bg-rose-700 p-2 rounded-lg">
                    <i data-lucide="alert-triangle" class="w-6 h-6 text-white"></i>
                </div>
                <div>
                    <h3 class="font-bold text-base">⚠️ เตือนภัย: ตารางงานเกินขีดจำกัด (งานล้นมือ)!</h3>
                    <p class="text-xs text-rose-100" id="overloadBannerText">คุณมีวันที่งานเกินพิกัดกำลังส่งผลต่อคุณภาพงาน กรุณาปรับลดหรือกระจายงาน</p>
                </div>
            </div>
            <button onclick="switchTab('dashboard')" class="bg-white text-rose-700 hover:bg-rose-50 text-xs font-bold px-3 py-2 rounded-lg transition shrink-0 shadow">
                ตรวจสอบจุดคอขวด
            </button>
        </div>

        <!-- VIEW 1: DASHBOARD & WORKLOAD ANALYSIS -->
        <div id="view-dashboard" class="space-y-6">
            <!-- Summary Metric Cards -->
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                    <div>
                        <p class="text-xs font-medium text-slate-500 mb-1">โปรเจกต์ที่ดำเนินอยู่</p>
                        <h3 class="text-2xl font-bold text-slate-800" id="statActiveProjects">0</h3>
                        <p class="text-xs text-slate-400 mt-1" id="statTotalProjects">จากทั้งหมด 0 โปรเจกต์</p>
                    </div>
                    <div class="w-12 h-12 bg-indigo-50 text-indigo-600 rounded-xl flex items-center justify-center">
                        <i data-lucide="folder" class="w-6 h-6"></i>
                    </div>
                </div>

                <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                    <div>
                        <p class="text-xs font-medium text-slate-500 mb-1">งานค้างทั้งหมด</p>
                        <h3 class="text-2xl font-bold text-slate-800" id="statOpenTasks">0</h3>
                        <p class="text-xs text-indigo-600 mt-1 font-medium" id="statTotalHours">0 ชั่วโมงโดยประมาณ</p>
                    </div>
                    <div class="w-12 h-12 bg-blue-50 text-blue-600 rounded-xl flex items-center justify-center">
                        <i data-lucide="check-square" class="w-6 h-6"></i>
                    </div>
                </div>

                <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                    <div>
                        <p class="text-xs font-medium text-slate-500 mb-1">ต้องติดตาม (Follow-up)</p>
                        <h3 class="text-2xl font-bold text-amber-600" id="statFollowupTasks">0</h3>
                        <p class="text-xs text-amber-600 mt-1">รอการตอบกลับ / คอขวด</p>
                    </div>
                    <div class="w-12 h-12 bg-amber-50 text-amber-600 rounded-xl flex items-center justify-center">
                        <i data-lucide="clock" class="w-6 h-6"></i>
                    </div>
                </div>

                <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between" id="overloadStatCard">
                    <div>
                        <p class="text-xs font-medium text-slate-500 mb-1">วันที่งานล้นมือ (> Max)</p>
                        <h3 class="text-2xl font-bold text-slate-800" id="statOverloadedDays">0 วัน</h3>
                        <p class="text-xs text-slate-400 mt-1" id="statOverloadStatus">อยู่ในเกณฑ์ปกติ</p>
                    </div>
                    <div id="overloadIconBox" class="w-12 h-12 bg-emerald-50 text-emerald-600 rounded-xl flex items-center justify-center">
                        <i data-lucide="shield-check" class="w-6 h-6"></i>
                    </div>
                </div>
            </div>

            <!-- Main Charts & Analysis Grid -->
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Daily Workload Capacity Chart (2 Cols) -->
                <div class="lg:col-span-2 bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
                    <div class="flex items-center justify-between border-b border-slate-100 pb-3">
                        <div>
                            <h2 class="text-base font-bold text-slate-800 flex items-center gap-2">
                                <i data-lucide="bar-chart-3" class="w-5 h-5 text-indigo-600"></i>
                                วิเคราะห์ภาระงานรายวัน (Workload per Day)
                            </h2>
                            <p class="text-xs text-slate-500">เปรียบเทียบชั่วโมงงานที่ต้องทำจริง กับขีดจำกัดกำลังการทำงานสูงสุดต่อวัน</p>
                        </div>
                        <span class="text-xs font-semibold px-2.5 py-1 rounded-full bg-slate-100 text-slate-600" id="chartDateRange">7 วันข้างหน้า</span>
                    </div>

                    <div class="relative h-64">
                        <canvas id="workloadChart"></canvas>
                    </div>
                </div>

                <!-- Project Workload Distribution (1 Col) -->
                <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4 flex flex-col justify-between">
                    <div>
                        <div class="border-b border-slate-100 pb-3 mb-4">
                            <h2 class="text-base font-bold text-slate-800 flex items-center gap-2">
                                <i data-lucide="pie-chart" class="w-5 h-5 text-indigo-600"></i>
                                สัดส่วนงานตามโปรเจกต์
                            </h2>
                            <p class="text-xs text-slate-500">การกระจายตัวของจำนวนชั่วโมงงานในแต่ละโปรเจกต์</p>
                        </div>
                        <div class="relative h-52 flex items-center justify-center">
                            <canvas id="projectPieChart"></canvas>
                        </div>
                    </div>
                    <div id="projectPieLegend" class="space-y-1.5 text-xs border-t border-slate-100 pt-3">
                        <!-- Dynamic Legend Inserted Here -->
                    </div>
                </div>
            </div>

            <!-- Overdue & Immediate Attention Tasks Section -->
            <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
                <div class="flex items-center justify-between border-b border-slate-100 pb-3">
                    <h2 class="text-base font-bold text-slate-800 flex items-center gap-2">
                        <i data-lucide="alert-circle" class="w-5 h-5 text-rose-500"></i>
                        งานที่เกินกำหนดส่ง / ต้องดำเนินการเร่งด่วน
                    </h2>
                    <span class="text-xs text-slate-500">เรียงตามความเร่งด่วน</span>
                </div>
                
                <div id="urgentDashboardList" class="divide-y divide-slate-100">
                    <!-- Dynamic List Inserted Here -->
                </div>
            </div>
        </div>

        <!-- VIEW 2: KANBAN BOARD -->
        <div id="view-kanban" class="hidden space-y-4">
            <!-- Kanban Filter Bar -->
            <div class="flex flex-wrap items-center justify-between gap-3 bg-white p-3 rounded-xl border border-slate-200">
                <div class="flex items-center gap-2">
                    <span class="text-xs font-semibold text-slate-500">กรองตามโปรเจกต์:</span>
                    <select id="kanbanProjectFilter" onchange="renderKanbanBoard()" class="text-xs border border-slate-300 rounded-lg px-2.5 py-1.5 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                        <option value="ALL">ทั้งหมด</option>
                    </select>
                </div>
                <div class="text-xs text-slate-500 flex items-center gap-2">
                    <span class="inline-block w-2.5 h-2.5 rounded-full bg-amber-400"></span> สีส้ม = ต้องติดตาม
                    <span class="inline-block w-2.5 h-2.5 rounded-full bg-rose-500 ml-2"></span> สีแดง = ด่วนที่สุด/ล้นมือ
                </div>
            </div>

            <!-- Columns Container -->
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4 items-start">
                <!-- Column: To Do -->
                <div class="bg-slate-100 rounded-2xl p-4 border border-slate-200 flex flex-col max-h-[calc(100vh-220px)]">
                    <div class="flex items-center justify-between mb-3 px-1">
                        <div class="flex items-center gap-2">
                            <span class="w-3 h-3 rounded-full bg-slate-400"></span>
                            <h3 class="font-bold text-sm text-slate-700">ยังไม่เริ่ม (To Do)</h3>
                        </div>
                        <span id="count-TODO" class="bg-slate-200 text-slate-700 text-xs px-2 py-0.5 rounded-full font-bold">0</span>
                    </div>
                    <div id="kanban-TODO" class="kanban-col space-y-3 overflow-y-auto pr-1 flex-1 min-h-[150px]">
                        <!-- Cards -->
                    </div>
                </div>

                <!-- Column: In Progress -->
                <div class="bg-blue-50/70 rounded-2xl p-4 border border-blue-100 flex flex-col max-h-[calc(100vh-220px)]">
                    <div class="flex items-center justify-between mb-3 px-1">
                        <div class="flex items-center gap-2">
                            <span class="w-3 h-3 rounded-full bg-blue-500"></span>
                            <h3 class="font-bold text-sm text-blue-900">กำลังทำ (In Progress)</h3>
                        </div>
                        <span id="count-IN_PROGRESS" class="bg-blue-100 text-blue-800 text-xs px-2 py-0.5 rounded-full font-bold">0</span>
                    </div>
                    <div id="kanban-IN_PROGRESS" class="kanban-col space-y-3 overflow-y-auto pr-1 flex-1 min-h-[150px]">
                        <!-- Cards -->
                    </div>
                </div>

                <!-- Column: Waiting / Follow-up -->
                <div class="bg-amber-50/70 rounded-2xl p-4 border border-amber-200 flex flex-col max-h-[calc(100vh-220px)]">
                    <div class="flex items-center justify-between mb-3 px-1">
                        <div class="flex items-center gap-2">
                            <span class="w-3 h-3 rounded-full bg-amber-500 animate-pulse"></span>
                            <h3 class="font-bold text-sm text-amber-900">ต้องติดตาม (Follow-up)</h3>
                        </div>
                        <span id="count-WAITING" class="bg-amber-200 text-amber-900 text-xs px-2 py-0.5 rounded-full font-bold">0</span>
                    </div>
                    <div id="kanban-WAITING" class="kanban-col space-y-3 overflow-y-auto pr-1 flex-1 min-h-[150px]">
                        <!-- Cards -->
                    </div>
                </div>

                <!-- Column: Done -->
                <div class="bg-emerald-50/70 rounded-2xl p-4 border border-emerald-100 flex flex-col max-h-[calc(100vh-220px)]">
                    <div class="flex items-center justify-between mb-3 px-1">
                        <div class="flex items-center gap-2">
                            <span class="w-3 h-3 rounded-full bg-emerald-500"></span>
                            <h3 class="font-bold text-sm text-emerald-900">เสร็จสิ้น (Done)</h3>
                        </div>
                        <span id="count-DONE" class="bg-emerald-100 text-emerald-800 text-xs px-2 py-0.5 rounded-full font-bold">0</span>
                    </div>
                    <div id="kanban-DONE" class="kanban-col space-y-3 overflow-y-auto pr-1 flex-1 min-h-[150px]">
                        <!-- Cards -->
                    </div>
                </div>
            </div>
        </div>

        <!-- VIEW 3: FOLLOW-UP TRACKER -->
        <div id="view-followup" class="hidden space-y-4">
            <div class="bg-amber-50 border border-amber-200 rounded-2xl p-5 flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                <div class="flex items-start gap-3">
                    <div class="p-2.5 bg-amber-500 text-white rounded-xl shadow shrink-0">
                        <i data-lucide="bell-ring" class="w-6 h-6"></i>
                    </div>
                    <div>
                        <h2 class="font-bold text-amber-950 text-base">ศูนย์รวมงานที่ต้องติดตาม (Follow-up Hub)</h2>
                        <p class="text-xs text-amber-800 mt-0.5">รวบรวมงานที่ติด Blockers, รอการตอบกลับจากเพื่อนร่วมงาน หรือบุคคลภายนอก ป้องกันการตกหล่น</p>
                    </div>
                </div>
            </div>

            <!-- Follow-up Table / Cards -->
            <div class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
                <div class="p-4 border-b border-slate-100 flex flex-wrap items-center justify-between gap-3">
                    <h3 class="font-bold text-sm text-slate-800">รายการงานที่รอดำเนินการติดตาม</h3>
                    <div class="text-xs text-slate-500" id="followupCountText">พบ 0 รายการ</div>
                </div>

                <div class="overflow-x-auto">
                    <table class="w-full text-left text-xs">
                        <thead class="bg-slate-50 text-slate-500 uppercase tracking-wider font-semibold border-b border-slate-200">
                            <tr>
                                <th class="py-3 px-4">ชื่องาน</th>
                                <th class="py-3 px-4">โปรเจกต์</th>
                                <th class="py-3 px-4">ผู้รับผิดชอบ/ผู้ต้องติดต่อ</th>
                                <th class="py-3 px-4">ความสำคัญ</th>
                                <th class="py-3 px-4">กำหนดส่ง</th>
                                <th class="py-3 px-4 text-right">การจัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="followupTableBody" class="divide-y divide-slate-100 text-slate-700">
                            <!-- Dynamic Content -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- VIEW 4: ALL TASKS TABLE -->
        <div id="view-alltasks" class="hidden space-y-4">
            <!-- Filter & Search Controls -->
            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex flex-col sm:flex-row items-center justify-between gap-3">
                <div class="relative w-full sm:w-72">
                    <i data-lucide="search" class="w-4 h-4 absolute left-3 top-3 text-slate-400"></i>
                    <input type="text" id="taskSearchInput" onkeyup="renderAllTasksTable()" placeholder="ค้นหางาน หรือชื่อผู้ติดต่อ..." class="w-full pl-9 pr-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>

                <div class="flex items-center gap-2 w-full sm:w-auto overflow-x-auto pb-1 sm:pb-0">
                    <select id="filterProject" onchange="renderAllTasksTable()" class="text-xs border border-slate-300 rounded-xl px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                        <option value="ALL">ทุกโปรเจกต์</option>
                    </select>

                    <select id="filterStatus" onchange="renderAllTasksTable()" class="text-xs border border-slate-300 rounded-xl px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                        <option value="ALL">ทุกสถานะ</option>
                        <option value="TODO">ยังไม่เริ่ม</option>
                        <option value="IN_PROGRESS">กำลังทำ</option>
                        <option value="WAITING">ต้องติดตาม</option>
                        <option value="DONE">เสร็จสิ้น</option>
                    </select>

                    <select id="filterPriority" onchange="renderAllTasksTable()" class="text-xs border border-slate-300 rounded-xl px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                        <option value="ALL">ทุกระดับความสำคัญ</option>
                        <option value="URGENT">ด่วนที่สุด</option>
                        <option value="HIGH">สูง</option>
                        <option value="MEDIUM">ปานกลาง</option>
                        <option value="LOW">ต่ำ</option>
                    </select>
                </div>
            </div>

            <!-- Task List Table -->
            <div class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
                <div class="overflow-x-auto">
                    <table class="w-full text-left text-xs">
                        <thead class="bg-slate-50 text-slate-500 uppercase tracking-wider font-semibold border-b border-slate-200">
                            <tr>
                                <th class="py-3 px-4">ชื่องาน</th>
                                <th class="py-3 px-4">โปรเจกต์</th>
                                <th class="py-3 px-4">สถานะ</th>
                                <th class="py-3 px-4">ความสำคัญ</th>
                                <th class="py-3 px-4">เวลาประเมิน</th>
                                <th class="py-3 px-4">กำหนดส่ง</th>
                                <th class="py-3 px-4">ผู้ติดต่อ/ติดตาม</th>
                                <th class="py-3 px-4 text-right">การจัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="allTasksTableBody" class="divide-y divide-slate-100 text-slate-700">
                            <!-- Dynamic Rows -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

    </div>

    <!-- MODAL 1: CREATE / EDIT TASK -->
    <div id="taskModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white w-full max-w-lg rounded-2xl shadow-2xl border border-slate-100 overflow-hidden flex flex-col max-h-[90vh]">
            <div class="bg-slate-900 text-white px-6 py-4 flex items-center justify-between">
                <h3 class="font-bold text-base flex items-center gap-2" id="taskModalTitle">
                    <i data-lucide="plus-circle" class="w-5 h-5 text-amber-400"></i> เพิ่มงานใหม่
                </h3>
                <button onclick="closeModal('taskModal')" class="text-slate-400 hover:text-white transition">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>

            <form id="taskForm" onsubmit="saveTask(event)" class="p-6 space-y-4 overflow-y-auto flex-1">
                <input type="hidden" id="taskId">

                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">ชื่องาน <span class="text-rose-500">*</span></label>
                    <input type="text" id="taskTitle" required placeholder="เช่น ออกแบบ UI หน้าแรก, ติดตามใบเสนอราคา" class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">โปรเจกต์ <span class="text-rose-500">*</span></label>
                        <select id="taskProject" required class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <!-- Options dynamically populated -->
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">สถานะงาน <span class="text-rose-500">*</span></label>
                        <select id="taskStatus" required class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="TODO">ยังไม่เริ่ม (To Do)</option>
                            <option value="IN_PROGRESS">กำลังทำ (In Progress)</option>
                            <option value="WAITING">ต้องติดตาม (Follow-up)</option>
                            <option value="DONE">เสร็จสิ้น (Done)</option>
                        </select>
                    </div>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">ระดับความสำคัญ</label>
                        <select id="taskPriority" class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="LOW">ต่ำ</option>
                            <option value="MEDIUM" selected>ปานกลาง</option>
                            <option value="HIGH">สูง</option>
                            <option value="URGENT">ด่วนที่สุด ⚠️</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">ประเมินเวลา (ชม.)</label>
                        <input type="number" id="taskHours" min="0.5" step="0.5" value="2" required class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">กำหนดส่ง (Due Date)</label>
                        <input type="date" id="taskDueDate" required class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">ผู้รับผิดชอบ / ผู้ที่ต้องติดต่อติดตาม</label>
                    <input type="text" id="taskAssignee" placeholder="เช่น คุณสมชาย (ฝ่ายขาย), ทีมออกแบบ External" class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">รายละเอียด / หมายเหตุติดตาม</label>
                    <textarea id="taskDescription" rows="3" placeholder="รายละเอียดของงาน หรือสิ่งที่ต้องติดตามผล..." class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500"></textarea>
                </div>

                <div class="pt-3 border-t border-slate-100 flex items-center justify-end gap-3">
                    <button type="button" onclick="closeModal('taskModal')" class="px-4 py-2 text-xs font-medium text-slate-600 hover:bg-slate-100 rounded-xl transition">ยกเลิก</button>
                    <button type="submit" class="px-5 py-2 text-xs font-bold bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl shadow transition">บันทึกข้อมูล</button>
                </div>
            </form>
        </div>
    </div>

    <!-- MODAL 2: CREATE / EDIT PROJECT -->
    <div id="projectModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white w-full max-w-md rounded-2xl shadow-2xl border border-slate-100 overflow-hidden">
            <div class="bg-slate-900 text-white px-6 py-4 flex items-center justify-between">
                <h3 class="font-bold text-base flex items-center gap-2">
                    <i data-lucide="folder-plus" class="w-5 h-5 text-indigo-400"></i> สร้างโปรเจกต์ใหม่
                </h3>
                <button onclick="closeModal('projectModal')" class="text-slate-400 hover:text-white transition">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>

            <form id="projectForm" onsubmit="saveProject(event)" class="p-6 space-y-4">
                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">ชื่อโปรเจกต์ <span class="text-rose-500">*</span></label>
                    <input type="text" id="projectName" required placeholder="เช่น พัฒนาเว็บไซต์บริษัท, แคมเปญการตลาด Q4" class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">ธีมสีประจำโปรเจกต์</label>
                    <div class="flex items-center gap-3">
                        <input type="color" id="projectColor" value="#4f46e5" class="w-10 h-10 rounded-lg cursor-pointer border-0">
                        <span class="text-xs text-slate-500">ใช้แยกแยะโปรเจกต์ในหน้าบอร์ดและกราฟ</span>
                    </div>
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">รายละเอียดโปรเจกต์</label>
                    <textarea id="projectDescription" rows="3" placeholder="อธิบายเป้าหมาย หรือขอบเขตงานโดยสังเขป..." class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500"></textarea>
                </div>

                <div class="pt-3 border-t border-slate-100 flex items-center justify-end gap-3">
                    <button type="button" onclick="closeModal('projectModal')" class="px-4 py-2 text-xs font-medium text-slate-600 hover:bg-slate-100 rounded-xl transition">ยกเลิก</button>
                    <button type="submit" class="px-5 py-2 text-xs font-bold bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl shadow transition">สร้างโปรเจกต์</button>
                </div>
            </form>
        </div>
    </div>

    <!-- JAVASCRIPT APP LOGIC -->
    <script>
        // --- DATA STATE & INITIALIZATION ---
        const LOCAL_STORAGE_KEY_TASKS = 'wkt_tasks_v1';
        const LOCAL_STORAGE_KEY_PROJECTS = 'wkt_projects_v1';
        const LOCAL_STORAGE_KEY_CAPACITY = 'wkt_capacity_v1';

        let dailyCapacity = parseInt(localStorage.getItem(LOCAL_STORAGE_KEY_CAPACITY)) || 8;
        let projects = JSON.parse(localStorage.getItem(LOCAL_STORAGE_KEY_PROJECTS)) || [];
        let tasks = JSON.parse(localStorage.getItem(LOCAL_STORAGE_KEY_TASKS)) || [];

        // Chart Instances
        let workloadChartInstance = null;
        let projectPieChartInstance = null;

        // Initialize App on Load
        window.onload = function() {
            lucide.createIcons();
            document.getElementById('dailyCapacityInput').value = dailyCapacity;

            // Load Sample Data if Empty
            if (projects.length === 0 && tasks.length === 0) {
                seedSampleData();
            }

            populateProjectDropdowns();
            renderDashboard();
            renderKanbanBoard();
            renderFollowupView();
            renderAllTasksTable();
        };

        // Seed Initial Thai Sample Data for quick testing
        function seedSampleData() {
            const today = new Date();
            const formatDate = (daysToAdd) => {
                const d = new Date(today);
                d.setDate(d.getDate() + daysToAdd);
                return d.toISOString().split('T')[0];
            };

            projects = [
                { id: 'p1', name: 'พัฒนาระบบ ERP สำหรับคลังสินค้า', color: '#4f46e5', description: 'โปรเจกต์ปรับปรุงระบบจัดการสต็อกสินค้า' },
                { id: 'p2', name: 'แคมเปญการตลาด Q4', color: '#059669', description: 'การโปรโมตสินค้าใหม่ต้อนรับเทศกาล' },
                { id: 'p3', name: 'จัดทำรายงานบัญชีประจำปี', color: '#d97706', description: 'สรุปงบการเงินและเตรียมยื่นภาษี' }
            ];

            tasks = [
                {
                    id: 't1',
                    title: 'ออกแบบ Database Schema สำหรับสินค้า',
                    projectId: 'p1',
                    status: 'IN_PROGRESS',
                    priority: 'HIGH',
                    hours: 5,
                    dueDate: formatDate(0), // Today
                    assignee: 'คุณปิติ (Tech Lead)',
                    description: 'ต้องรอยืนยันโครงสร้างฟิลด์จากทีมบัญชีก่อน'
                },
                {
                    id: 't2',
                    title: 'ทดสอบ Integration API กับระบบขนส่ง',
                    projectId: 'p1',
                    status: 'TODO',
                    priority: 'URGENT',
                    hours: 6,
                    dueDate: formatDate(0), // Today (Overloading today: 5+6 = 11 hrs > 8 hrs capacity!)
                    assignee: 'ตนเอง',
                    description: 'งานด่วนเพื่อเตรียมปล่อย Demo'
                },
                {
                    id: 't3',
                    title: 'ตามใบเสนอราคาจากเอเจนซี่โฆษณา',
                    projectId: 'p2',
                    status: 'WAITING',
                    priority: 'MEDIUM',
                    hours: 2,
                    dueDate: formatDate(-1), // Overdue Follow-up
                    assignee: 'คุณนิภา (Agency Contact)',
                    description: 'ส่งอีเมลไปเมื่อวันก่อน ยังไม่ได้รับการตอบกลับ'
                },
                {
                    id: 't4',
                    title: 'อนุมัติ Artwork แบนเนอร์ Facebook/IG',
                    projectId: 'p2',
                    status: 'WAITING',
                    priority: 'HIGH',
                    hours: 3,
                    dueDate: formatDate(1),
                    assignee: 'คุณสมศักดิ์ (Graphic Designer)',
                    description: 'รอแก้งานรอบสุดท้าย'
                },
                {
                    id: 't5',
                    title: 'รวบรวมเอกสารภาษีซื้อ-ขายเดือนกันยายน',
                    projectId: 'p3',
                    status: 'IN_PROGRESS',
                    priority: 'MEDIUM',
                    hours: 4,
                    dueDate: formatDate(2),
                    assignee: 'คุณเมย์ (ฝ่ายบัญชี)',
                    description: 'กำลังตรวจสอบความถูกต้องของใบกำกับภาษี'
                },
                {
                    id: 't6',
                    title: 'สรุปรายงานผลการดำเนินงานไตรมาส 3',
                    projectId: 'p3',
                    status: 'DONE',
                    priority: 'LOW',
                    hours: 3,
                    dueDate: formatDate(-3),
                    assignee: 'ตนเอง',
                    description: 'ส่งผู้บริหารเรียบร้อยแล้ว'
                }
            ];

            saveDataToLocalStorage();
        }

        // Save State
        function saveDataToLocalStorage() {
            localStorage.setItem(LOCAL_STORAGE_KEY_PROJECTS, JSON.stringify(projects));
            localStorage.setItem(LOCAL_STORAGE_KEY_TASKS, JSON.stringify(tasks));
            localStorage.setItem(LOCAL_STORAGE_KEY_CAPACITY, dailyCapacity.toString());
        }

        // Update Daily Capacity Limit
        function updateCapacity(val) {
            dailyCapacity = parseInt(val) || 8;
            saveDataToLocalStorage();
            renderDashboard();
        }

        // Tab Switching Logic
        function switchTab(tabName) {
            const tabs = ['dashboard', 'kanban', 'followup', 'alltasks'];
            tabs.forEach(t => {
                const view = document.getElementById(`view-${t}`);
                const btn = document.getElementById(`tab-${t}`);
                if (t === tabName) {
                    view.classList.remove('hidden');
                    btn.classList.add('bg-white', 'text-indigo-900', 'shadow-sm');
                    btn.classList.remove('text-slate-600');
                } else {
                    view.classList.add('hidden');
                    btn.classList.remove('bg-white', 'text-indigo-900', 'shadow-sm');
                    btn.classList.add('text-slate-600');
                }
            });

            // Re-render specifics when tab changes
            if (tabName === 'dashboard') renderDashboard();
            if (tabName === 'kanban') renderKanbanBoard();
            if (tabName === 'followup') renderFollowupView();
            if (tabName === 'alltasks') renderAllTasksTable();
            lucide.createIcons();
        }

        // Modal Handlers
        function openModal(modalId, taskIdToEdit = null) {
            document.getElementById(modalId).classList.remove('hidden');
            if (modalId === 'taskModal') {
                if (taskIdToEdit) {
                    const task = tasks.find(t => t.id === taskIdToEdit);
                    if (task) {
                        document.getElementById('taskModalTitle').innerText = 'แก้ไขรายการงาน';
                        document.getElementById('taskId').value = task.id;
                        document.getElementById('taskTitle').value = task.title;
                        document.getElementById('taskProject').value = task.projectId;
                        document.getElementById('taskStatus').value = task.status;
                        document.getElementById('taskPriority').value = task.priority;
                        document.getElementById('taskHours').value = task.hours;
                        document.getElementById('taskDueDate').value = task.dueDate;
                        document.getElementById('taskAssignee').value = task.assignee || '';
                        document.getElementById('taskDescription').value = task.description || '';
                    }
                } else {
                    document.getElementById('taskModalTitle').innerText = 'เพิ่มงานใหม่';
                    document.getElementById('taskForm').reset();
                    document.getElementById('taskId').value = '';
                    document.getElementById('taskDueDate').value = new Date().toISOString().split('T')[0];
                }
            }
            lucide.createIcons();
        }

        function closeModal(modalId) {
            document.getElementById(modalId).classList.add('hidden');
        }

        // Populate Select Dropdowns
        function populateProjectDropdowns() {
            const taskProjectSelect = document.getElementById('taskProject');
            const filterProjectSelect = document.getElementById('filterProject');
            const kanbanProjectFilter = document.getElementById('kanbanProjectFilter');

            let optionsHtml = projects.map(p => `<option value="${p.id}">${p.name}</option>`).join('');
            taskProjectSelect.innerHTML = optionsHtml;

            filterProjectSelect.innerHTML = `<option value="ALL">ทุกโปรเจกต์</option>` + optionsHtml;
            kanbanProjectFilter.innerHTML = `<option value="ALL">ทั้งหมด</option>` + optionsHtml;
        }

        // --- PROJECT ACTIONS ---
        function saveProject(e) {
            e.preventDefault();
            const name = document.getElementById('projectName').value.trim();
            const color = document.getElementById('projectColor').value;
            const description = document.getElementById('projectDescription').value.trim();

            if (!name) return;

            const newProj = {
                id: 'p_' + Date.now(),
                name,
                color,
                description
            };

            projects.push(newProj);
            saveDataToLocalStorage();
            populateProjectDropdowns();
            closeModal('projectModal');
            document.getElementById('projectForm').reset();
            renderDashboard();
        }

        // --- TASK ACTIONS ---
        function saveTask(e) {
            e.preventDefault();
            const id = document.getElementById('taskId').value;
            const title = document.getElementById('taskTitle').value.trim();
            const projectId = document.getElementById('taskProject').value;
            const status = document.getElementById('taskStatus').value;
            const priority = document.getElementById('taskPriority').value;
            const hours = parseFloat(document.getElementById('taskHours').value) || 1;
            const dueDate = document.getElementById('taskDueDate').value;
            const assignee = document.getElementById('taskAssignee').value.trim();
            const description = document.getElementById('taskDescription').value.trim();

            if (!title || !projectId || !dueDate) return;

            if (id) {
                // Update
                const index = tasks.findIndex(t => t.id === id);
                if (index !== -1) {
                    tasks[index] = { id, title, projectId, status, priority, hours, dueDate, assignee, description };
                }
            } else {
                // Create
                const newTask = {
                    id: 't_' + Date.now(),
                    title,
                    projectId,
                    status,
                    priority,
                    hours,
                    dueDate,
                    assignee,
                    description
                };
                tasks.push(newTask);
            }

            saveDataToLocalStorage();
            closeModal('taskModal');
            renderDashboard();
            renderKanbanBoard();
            renderFollowupView();
            renderAllTasksTable();
        }

        function updateTaskStatus(taskId, newStatus) {
            const task = tasks.find(t => t.id === taskId);
            if (task) {
                task.status = newStatus;
                saveDataToLocalStorage();
                renderDashboard();
                renderKanbanBoard();
                renderFollowupView();
                renderAllTasksTable();
            }
        }

        function deleteTask(taskId) {
            if (confirm('คุณแน่ใจหรือไม่ว่าต้องการลบรายการงานนี้?')) {
                tasks = tasks.filter(t => t.id !== taskId);
                saveDataToLocalStorage();
                renderDashboard();
                renderKanbanBoard();
                renderFollowupView();
                renderAllTasksTable();
            }
        }

        // --- VIEW 1: DASHBOARD & WORKLOAD LOGIC ---
        function renderDashboard() {
            // Active Tasks (Exclude Done)
            const openTasks = tasks.filter(t => t.status !== 'DONE');
            const followupTasks = tasks.filter(t => t.status === 'WAITING');
            const totalHours = openTasks.reduce((acc, t) => acc + t.hours, 0);

            // Calculate Overloaded Days in the upcoming 7 days
            const dailyWorkloadMap = {};
            const today = new Date();
            today.setHours(0,0,0,0);

            // Group active hours by due date
            openTasks.forEach(t => {
                if (!t.dueDate) return;
                dailyWorkloadMap[t.dueDate] = (dailyWorkloadMap[t.dueDate] || 0) + t.hours;
            });

            let overloadedDaysCount = 0;
            Object.keys(dailyWorkloadMap).forEach(dateStr => {
                if (dailyWorkloadMap[dateStr] > dailyCapacity) {
                    overloadedDaysCount++;
                }
            });

            // Update Metric Display
            document.getElementById('statActiveProjects').innerText = projects.length;
            document.getElementById('statTotalProjects').innerText = `จากทั้งหมด ${projects.length} โปรเจกต์`;
            
            document.getElementById('statOpenTasks').innerText = openTasks.length;
            document.getElementById('statTotalHours').innerText = `${totalHours} ชั่วโมงโดยประมาณ`;

            document.getElementById('statFollowupTasks').innerText = followupTasks.length;
            document.getElementById('followupBadge').innerText = followupTasks.length;
            if (followupTasks.length > 0) {
                document.getElementById('followupBadge').classList.remove('hidden');
            } else {
                document.getElementById('followupBadge').classList.add('hidden');
            }

            document.getElementById('statOverloadedDays').innerText = `${overloadedDaysCount} วัน`;
            
            const overloadBanner = document.getElementById('overloadBanner');
            const overloadIconBox = document.getElementById('overloadIconBox');
            const overloadStatCard = document.getElementById('overloadStatCard');
            const statOverloadStatus = document.getElementById('statOverloadStatus');

            if (overloadedDaysCount > 0) {
                overloadBanner.classList.remove('hidden');
                document.getElementById('overloadBannerText').innerText = `พบ ${overloadedDaysCount} วันที่มีชั่วโมงงานสะสมเกิน ${dailyCapacity} ชม./วัน อาจเกิดภาวะงานล้นมือ!`;
                
                overloadIconBox.className = "w-12 h-12 bg-rose-100 text-rose-600 rounded-xl flex items-center justify-center";
                overloadIconBox.innerHTML = `<i data-lucide="alert-triangle" class="w-6 h-6"></i>`;
                statOverloadStatus.innerText = "⚠️ ภาระงานเกินขีดจำกัด!";
                statOverloadStatus.className = "text-xs text-rose-600 font-bold mt-1";
            } else {
                overloadBanner.classList.add('hidden');
                overloadIconBox.className = "w-12 h-12 bg-emerald-50 text-emerald-600 rounded-xl flex items-center justify-center";
                overloadIconBox.innerHTML = `<i data-lucide="shield-check" class="w-6 h-6"></i>`;
                statOverloadStatus.innerText = "อยู่ในเกณฑ์ปกติ";
                statOverloadStatus.className = "text-xs text-emerald-600 font-medium mt-1";
            }

            // Render Urgent / Overdue Tasks List
            renderUrgentDashboardList();

            // Render Charts
            renderWorkloadBarChart(dailyWorkloadMap);
            renderProjectPieChart();

            lucide.createIcons();
        }

        function renderUrgentDashboardList() {
            const container = document.getElementById('urgentDashboardList');
            const todayStr = new Date().toISOString().split('T')[0];

            // Filter Overdue or Urgent tasks (Exclude DONE)
            const urgentTasks = tasks.filter(t => {
                if (t.status === 'DONE') return false;
                const isOverdue = t.dueDate < todayStr;
                const isUrgent = t.priority === 'URGENT';
                return isOverdue || isUrgent || t.status === 'WAITING';
            }).sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate));

            if (urgentTasks.length === 0) {
                container.innerHTML = `
                    <div class="py-8 text-center text-slate-400 text-xs">
                        <i data-lucide="check-circle-2" class="w-8 h-8 mx-auto mb-2 text-emerald-500"></i>
                        ไม่มีงานด่วนหรือตกค้างในขณะนี้ ระบบทำงานได้เรียบร้อยดี!
                    </div>
                `;
                return;
            }

            container.innerHTML = urgentTasks.map(t => {
                const proj = projects.find(p => p.id === t.projectId) || { name: 'ไม่มีโปรเจกต์', color: '#64748b' };
                const isOverdue = t.dueDate < todayStr;
                
                let badgeClass = "bg-slate-100 text-slate-700";
                let badgeText = t.priority;
                if (t.priority === 'URGENT') { badgeClass = "bg-rose-100 text-rose-700 font-bold"; badgeText = "ด่วนที่สุด ⚠️"; }
                if (isOverdue) { badgeClass = "bg-rose-600 text-white font-bold"; badgeText = "เกินกำหนดส่ง!"; }

                return `
                    <div class="py-3 flex flex-col sm:flex-row sm:items-center justify-between gap-2 hover:bg-slate-50 px-2 rounded-xl transition">
                        <div class="flex items-start gap-3">
                            <span class="w-2.5 h-2.5 rounded-full mt-1.5 shrink-0" style="background-color: ${proj.color}"></span>
                            <div>
                                <div class="flex items-center gap-2">
                                    <h4 class="font-semibold text-xs text-slate-800">${escapeHtml(t.title)}</h4>
                                    <span class="text-[10px] px-2 py-0.5 rounded-full ${badgeClass}">${badgeText}</span>
                                </div>
                                <div class="text-[11px] text-slate-500 mt-0.5 flex flex-wrap items-center gap-x-3">
                                    <span>โปรเจกต์: <strong>${escapeHtml(proj.name)}</strong></span>
                                    <span>กำหนด: <strong class="${isOverdue ? 'text-rose-600 font-bold' : ''}">${t.dueDate}</strong></span>
                                    ${t.assignee ? `<span>ผู้ติดต่อ: <strong>${escapeHtml(t.assignee)}</strong></span>` : ''}
                                </div>
                            </div>
                        </div>

                        <div class="flex items-center gap-2 self-end sm:self-center">
                            <select onchange="updateTaskStatus('${t.id}', this.value)" class="text-[11px] border border-slate-300 rounded-lg px-2 py-1 bg-white focus:outline-none focus:ring-1 focus:ring-indigo-500">
                                <option value="TODO" ${t.status === 'TODO' ? 'selected' : ''}>ยังไม่เริ่ม</option>
                                <option value="IN_PROGRESS" ${t.status === 'IN_PROGRESS' ? 'selected' : ''}>กำลังทำ</option>
                                <option value="WAITING" ${t.status === 'WAITING' ? 'selected' : ''}>ต้องติดตาม</option>
                                <option value="DONE" ${t.status === 'DONE' ? 'selected' : ''}>เสร็จสิ้น</option>
                            </select>
                            <button onclick="openModal('taskModal', '${t.id}')" class="p-1 text-slate-400 hover:text-indigo-600 rounded">
                                <i data-lucide="edit-3" class="w-4 h-4"></i>
                            </button>
                        </div>
                    </div>
                `;
            }).join('');
        }

        // Render Chart JS Bar Chart for Daily Workload
        function renderWorkloadBarChart(dailyWorkloadMap) {
            const ctx = document.getElementById('workloadChart').getContext('2d');

            // Generate Next 7 Days Labels
            const labels = [];
            const dataPoints = [];
            const backgroundColors = [];
            const borderColors = [];

            const today = new Date();
            for (let i = 0; i < 7; i++) {
                const d = new Date(today);
                d.setDate(d.getDate() + i);
                const dateStr = d.toISOString().split('T')[0];
                
                // Format label in Thai date (e.g., 20 ต.ค.)
                const dayName = d.toLocaleDateString('th-TH', { weekday: 'short', day: 'numeric', month: 'short' });
                labels.push(dayName);

                const hours = dailyWorkloadMap[dateStr] || 0;
                dataPoints.push(hours);

                if (hours > dailyCapacity) {
                    backgroundColors.push('rgba(244, 63, 94, 0.7)'); // Rose Overload
                    borderColors.push('rgb(225, 29, 72)');
                } else if (hours > 0) {
                    backgroundColors.push('rgba(99, 102, 241, 0.7)'); // Indigo normal
                    borderColors.push('rgb(79, 70, 229)');
                } else {
                    backgroundColors.push('rgba(203, 213, 225, 0.4)'); // Empty
                    borderColors.push('rgb(148, 163, 184)');
                }
            }

            if (workloadChartInstance) {
                workloadChartInstance.destroy();
            }

            workloadChartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [
                        {
                            label: 'ชั่วโมงงานที่ค้างส่ง (ชม.)',
                            data: dataPoints,
                            backgroundColor: backgroundColors,
                            borderColor: borderColors,
                            borderWidth: 1.5,
                            borderRadius: 8
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const val = context.raw;
                                    const status = val > dailyCapacity ? ' ⚠️ (งานล้นมือ!)' : '';
                                    return `ภาระงาน: ${val} ชั่วโมง / ขีดจำกัด ${dailyCapacity} ชม.${status}`;
                                }
                            }
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            suggestedMax: dailyCapacity + 4,
                            grid: { color: '#f1f5f9' },
                            ticks: {
                                callback: function(value) { return value + ' ชม.'; }
                            }
                        },
                        x: {
                            grid: { display: false }
                        }
                    }
                }
            });
        }

        // Render Chart JS Pie Chart for Project Distribution
        function renderProjectPieChart() {
            const ctx = document.getElementById('projectPieChart').getContext('2d');
            const legendContainer = document.getElementById('projectPieLegend');

            // Calculate hours per project for open tasks
            const openTasks = tasks.filter(t => t.status !== 'DONE');
            const projectHours = {};
            
            projects.forEach(p => { projectHours[p.id] = 0; });
            openTasks.forEach(t => {
                if (projectHours[t.projectId] !== undefined) {
                    projectHours[t.projectId] += t.hours;
                }
            });

            const labels = [];
            const data = [];
            const colors = [];

            projects.forEach(p => {
                const hrs = projectHours[p.id] || 0;
                if (hrs > 0) {
                    labels.push(p.name);
                    data.push(hrs);
                    colors.push(p.color);
                }
            });

            if (projectPieChartInstance) {
                projectPieChartInstance.destroy();
            }

            if (data.length === 0) {
                legendContainer.innerHTML = `<p class="text-center text-slate-400 py-2">ไม่มีข้อมูลงานค้างในโปรเจกต์</p>`;
                return;
            }

            projectPieChartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: data,
                        backgroundColor: colors,
                        borderWidth: 2,
                        borderColor: '#ffffff'
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false }
                    },
                    cutout: '70%'
                }
            });

            // Build Custom Legend
            const totalHrs = data.reduce((a, b) => a + b, 0);
            legendContainer.innerHTML = projects.map(p => {
                const hrs = projectHours[p.id] || 0;
                if (hrs === 0) return '';
                const pct = Math.round((hrs / totalHrs) * 100);
                return `
                    <div class="flex items-center justify-between">
                        <div class="flex items-center gap-2 truncate">
                            <span class="w-2.5 h-2.5 rounded-full shrink-0" style="background-color: ${p.color}"></span>
                            <span class="truncate font-medium text-slate-700">${escapeHtml(p.name)}</span>
                        </div>
                        <span class="font-bold text-slate-800 shrink-0 ml-2">${hrs} ชม. (${pct}%)</span>
                    </div>
                `;
            }).join('');
        }

        // --- VIEW 2: KANBAN BOARD LOGIC ---
        function renderKanbanBoard() {
            const filterProjId = document.getElementById('kanbanProjectFilter').value;
            const statuses = ['TODO', 'IN_PROGRESS', 'WAITING', 'DONE'];

            statuses.forEach(status => {
                const colContainer = document.getElementById(`kanban-${status}`);
                const countBadge = document.getElementById(`count-${status}`);

                let filtered = tasks.filter(t => t.status === status);
                if (filterProjId !== 'ALL') {
                    filtered = filtered.filter(t => t.projectId === filterProjId);
                }

                countBadge.innerText = filtered.length;

                if (filtered.length === 0) {
                    colContainer.innerHTML = `
                        <div class="h-28 border-2 border-dashed border-slate-200 rounded-xl flex items-center justify-center text-slate-400 text-xs">
                            ไม่มีงานในช่องนี้
                        </div>
                    `;
                    return;
                }

                colContainer.innerHTML = filtered.map(t => {
                    const proj = projects.find(p => p.id === t.projectId) || { name: 'ทั่วไป', color: '#64748b' };
                    
                    let priorityBadge = "";
                    if (t.priority === 'URGENT') priorityBadge = `<span class="bg-rose-100 text-rose-700 font-bold px-2 py-0.5 rounded text-[10px]">ด่วนที่สุด ⚠️</span>`;
                    else if (t.priority === 'HIGH') priorityBadge = `<span class="bg-amber-100 text-amber-800 px-2 py-0.5 rounded text-[10px]">สูง</span>`;

                    const isWaiting = t.status === 'WAITING';

                    return `
                        <div class="bg-white p-3.5 rounded-xl border ${isWaiting ? 'border-amber-300 ring-1 ring-amber-300 shadow-amber-100' : 'border-slate-200'} shadow-sm space-y-2.5 hover:shadow-md transition">
                            <div class="flex items-start justify-between gap-2">
                                <span class="text-[10px] font-semibold px-2 py-0.5 rounded-full text-white truncate max-w-[130px]" style="background-color: ${proj.color}">
                                    ${escapeHtml(proj.name)}
                                </span>
                                ${priorityBadge}
                            </div>

                            <h4 class="font-bold text-xs text-slate-800 leading-snug">${escapeHtml(t.title)}</h4>

                            ${t.description ? `<p class="text-[11px] text-slate-500 line-clamp-2">${escapeHtml(t.description)}</p>` : ''}

                            <div class="pt-2 border-t border-slate-100 flex items-center justify-between text-[11px] text-slate-500">
                                <div class="flex items-center gap-1.5">
                                    <i data-lucide="clock" class="w-3.5 h-3.5 text-slate-400"></i>
                                    <span>${t.hours} ชม.</span>
                                </div>
                                <div class="flex items-center gap-1 text-slate-600 font-medium">
                                    <i data-lucide="calendar" class="w-3.5 h-3.5 text-slate-400"></i>
                                    <span>${t.dueDate}</span>
                                </div>
                            </div>

                            ${t.assignee ? `
                                <div class="bg-slate-50 px-2 py-1 rounded text-[10px] text-slate-600 flex items-center gap-1 truncate">
                                    <i data-lucide="user" class="w-3 h-3 text-slate-400 shrink-0"></i>
                                    <span class="truncate">ติด/ติดตาม: <strong>${escapeHtml(t.assignee)}</strong></span>
                                </div>
                            ` : ''}

                            <!-- Quick Action Dropdown -->
                            <div class="flex items-center justify-between pt-1">
                                <select onchange="updateTaskStatus('${t.id}', this.value)" class="text-[10px] border border-slate-200 rounded px-1.5 py-0.5 bg-slate-50 text-slate-700 focus:outline-none">
                                    <option value="TODO" ${t.status === 'TODO' ? 'selected' : ''}>ย้ายไป: To Do</option>
                                    <option value="IN_PROGRESS" ${t.status === 'IN_PROGRESS' ? 'selected' : ''}>ย้ายไป: In Progress</option>
                                    <option value="WAITING" ${t.status === 'WAITING' ? 'selected' : ''}>ย้ายไป: Follow-up</option>
                                    <option value="DONE" ${t.status === 'DONE' ? 'selected' : ''}>ย้ายไป: Done</option>
                                </select>

                                <div class="flex items-center gap-1">
                                    <button onclick="openModal('taskModal', '${t.id}')" class="p-1 text-slate-400 hover:text-indigo-600 rounded">
                                        <i data-lucide="edit-2" class="w-3.5 h-3.5"></i>
                                    </button>
                                    <button onclick="deleteTask('${t.id}')" class="p-1 text-slate-400 hover:text-rose-600 rounded">
                                        <i data-lucide="trash-2" class="w-3.5 h-3.5"></i>
                                    </button>
                                </div>
                            </div>
                        </div>
                    `;
                }).join('');
            });

            lucide.createIcons();
        }

        // --- VIEW 3: FOLLOW-UP HUB LOGIC ---
        function renderFollowupView() {
            const tableBody = document.getElementById('followupTableBody');
            const followupTasks = tasks.filter(t => t.status === 'WAITING');
            document.getElementById('followupCountText').innerText = `พบทั้งหมด ${followupTasks.length} รายการ`;

            if (followupTasks.length === 0) {
                tableBody.innerHTML = `
                    <tr>
                        <td colspan="6" class="py-12 text-center text-slate-400">
                            <i data-lucide="bell-off" class="w-8 h-8 mx-auto mb-2 text-slate-300"></i>
                            ไม่มีรายการที่ต้องติดตามในขณะนี้
                        </td>
                    </tr>
                `;
                lucide.createIcons();
                return;
            }

            const todayStr = new Date().toISOString().split('T')[0];

            tableBody.innerHTML = followupTasks.map(t => {
                const proj = projects.find(p => p.id === t.projectId) || { name: 'ทั่วไป', color: '#64748b' };
                const isOverdue = t.dueDate < todayStr;

                return `
                    <tr class="hover:bg-amber-50/50 transition">
                        <td class="py-3 px-4 font-bold text-slate-800">
                            ${escapeHtml(t.title)}
                            ${t.description ? `<p class="text-[11px] font-normal text-slate-500 truncate max-w-xs mt-0.5">${escapeHtml(t.description)}</p>` : ''}
                        </td>
                        <td class="py-3 px-4">
                            <span class="inline-flex items-center gap-1.5 px-2 py-0.5 rounded-full text-[10px] font-medium text-white" style="background-color: ${proj.color}">
                                ${escapeHtml(proj.name)}
                            </span>
                        </td>
                        <td class="py-3 px-4 font-semibold text-slate-700">
                            <div class="flex items-center gap-1">
                                <i data-lucide="user-check" class="w-3.5 h-3.5 text-amber-600"></i>
                                ${escapeHtml(t.assignee || 'ไม่ได้ระบุ')}
                            </div>
                        </td>
                        <td class="py-3 px-4">${getPriorityBadgeHtml(t.priority)}</td>
                        <td class="py-3 px-4 font-medium ${isOverdue ? 'text-rose-600 font-bold' : 'text-slate-600'}">
                            ${t.dueDate} ${isOverdue ? '⚠️ เกินกำหนด' : ''}
                        </td>
                        <td class="py-3 px-4 text-right">
                            <div class="flex items-center justify-end gap-2">
                                <button onclick="updateTaskStatus('${t.id}', 'DONE')" class="bg-emerald-600 hover:bg-emerald-700 text-white px-2.5 py-1 rounded-lg text-[11px] font-bold flex items-center gap-1 shadow-sm transition">
                                    <i data-lucide="check" class="w-3 h-3"></i> ทำเสร็จแล้ว
                                </button>
                                <button onclick="openModal('taskModal', '${t.id}')" class="p-1.5 text-slate-500 hover:text-indigo-600 hover:bg-slate-100 rounded-lg">
                                    <i data-lucide="edit-3" class="w-4 h-4"></i>
                                </button>
                            </div>
                        </td>
                    </tr>
                `;
            }).join('');

            lucide.createIcons();
        }

        // --- VIEW 4: ALL TASKS TABLE LOGIC ---
        function renderAllTasksTable() {
            const tableBody = document.getElementById('allTasksTableBody');
            const searchKeyword = document.getElementById('taskSearchInput').value.toLowerCase().trim();
            const projFilter = document.getElementById('filterProject').value;
            const statusFilter = document.getElementById('filterStatus').value;
            const priorityFilter = document.getElementById('filterPriority').value;

            let filtered = tasks.filter(t => {
                const matchSearch = t.title.toLowerCase().includes(searchKeyword) || (t.assignee && t.assignee.toLowerCase().includes(searchKeyword));
                const matchProj = projFilter === 'ALL' || t.projectId === projFilter;
                const matchStatus = statusFilter === 'ALL' || t.status === statusFilter;
                const matchPriority = priorityFilter === 'ALL' || t.priority === priorityFilter;

                return matchSearch && matchProj && matchStatus && matchPriority;
            });

            if (filtered.length === 0) {
                tableBody.innerHTML = `
                    <tr>
                        <td colspan="8" class="py-12 text-center text-slate-400">
                            ไม่พบข้อมูลรายการงานตามเงื่อนไขที่ค้นหา
                        </td>
                    </tr>
                `;
                return;
            }

            tableBody.innerHTML = filtered.map(t => {
                const proj = projects.find(p => p.id === t.projectId) || { name: 'ทั่วไป', color: '#64748b' };
                
                return `
                    <tr class="hover:bg-slate-50 transition">
                        <td class="py-3 px-4 font-semibold text-slate-800">${escapeHtml(t.title)}</td>
                        <td class="py-3 px-4">
                            <span class="inline-block px-2 py-0.5 rounded text-[10px] text-white font-medium" style="background-color: ${proj.color}">
                                ${escapeHtml(proj.name)}
                            </span>
                        </td>
                        <td class="py-3 px-4">${getStatusBadgeHtml(t.status)}</td>
                        <td class="py-3 px-4">${getPriorityBadgeHtml(t.priority)}</td>
                        <td class="py-3 px-4 font-medium">${t.hours} ชม.</td>
                        <td class="py-3 px-4">${t.dueDate}</td>
                        <td class="py-3 px-4 text-slate-600">${escapeHtml(t.assignee || '-')}</td>
                        <td class="py-3 px-4 text-right">
                            <div class="flex items-center justify-end gap-1">
                                <button onclick="openModal('taskModal', '${t.id}')" class="p-1.5 text-slate-400 hover:text-indigo-600 rounded-lg">
                                    <i data-lucide="edit-2" class="w-4 h-4"></i>
                                </button>
                                <button onclick="deleteTask('${t.id}')" class="p-1.5 text-slate-400 hover:text-rose-600 rounded-lg">
                                    <i data-lucide="trash-2" class="w-4 h-4"></i>
                                </button>
                            </div>
                        </td>
                    </tr>
                `;
            }).join('');

            lucide.createIcons();
        }

        // --- HELPER BADGE FUNCTIONS ---
        function getStatusBadgeHtml(status) {
            switch (status) {
                case 'TODO': return `<span class="bg-slate-100 text-slate-700 px-2 py-0.5 rounded-full text-[10px] font-medium">ยังไม่เริ่ม</span>`;
                case 'IN_PROGRESS': return `<span class="bg-blue-100 text-blue-800 px-2 py-0.5 rounded-full text-[10px] font-medium">กำลังทำ</span>`;
                case 'WAITING': return `<span class="bg-amber-100 text-amber-900 px-2 py-0.5 rounded-full text-[10px] font-bold">ต้องติดตาม 🔔</span>`;
                case 'DONE': return `<span class="bg-emerald-100 text-emerald-800 px-2 py-0.5 rounded-full text-[10px] font-medium">เสร็จสิ้น</span>`;
                default: return '';
            }
        }

        function getPriorityBadgeHtml(priority) {
            switch (priority) {
                case 'URGENT': return `<span class="bg-rose-100 text-rose-700 font-bold px-2 py-0.5 rounded text-[10px]">ด่วนที่สุด ⚠️</span>`;
                case 'HIGH': return `<span class="bg-amber-100 text-amber-800 font-medium px-2 py-0.5 rounded text-[10px]">สูง</span>`;
                case 'MEDIUM': return `<span class="bg-slate-100 text-slate-700 font-medium px-2 py-0.5 rounded text-[10px]">ปานกลาง</span>`;
                case 'LOW': return `<span class="bg-slate-100 text-slate-500 font-medium px-2 py-0.5 rounded text-[10px]">ต่ำ</span>`;
                default: return '';
            }
        }

        // Utility to prevent XSS
        function escapeHtml(str) {
            if (!str) return '';
            return str.replace(/&/g, "&amp;")
                      .replace(/</g, "&lt;")
                      .replace(/>/g, "&gt;")
                      .replace(/"/g, "&quot;")
                      .replace(/'/g, "&#039;");
        }
    </script>
</body>
</html>