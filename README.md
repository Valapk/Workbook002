<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Work Book</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        
        body { font-family: 'Inter', sans-serif; background-color: #f3f4f6; -webkit-tap-highlight-color: transparent; }
        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #c7c7c7; border-radius: 10px; }

        .glass-panel { background: rgba(255, 255, 255, 0.95); backdrop-filter: blur(10px); }
        .card-hover { transition: all 0.3s ease; }
        .card-hover:hover { transform: translateY(-2px); box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1); }

        .slide-down { max-height: 0; overflow: hidden; transition: all 0.4s ease; opacity: 0; }
        .slide-down.open { max-height: 2000px; opacity: 1; }

        .history-row { display: grid; grid-template-columns: 70px 1fr 60px 60px; gap: 8px; align-items: center; padding: 10px; border-bottom: 1px solid #f3f4f6; }
        .history-row:hover { background-color: #f9fafb; }

        .settled-badge { background: linear-gradient(135deg, #10b981 0%, #059669 100%); color: white; padding: 4px 12px; border-radius: 20px; font-size: 11px; font-weight: bold; display: inline-block; margin-left: 8px; }
        
        .summary-box { background: white; border-radius: 12px; padding: 15px; margin-top: 15px; border: 1px solid #e5e7eb; }
        
        .lang-btn { padding: 6px 12px; border-radius: 20px; border: 1px solid #e5e7eb; background: white; font-size: 12px; cursor: pointer; }
        .lang-btn.active { background: #3b82f6; color: white; border-color: #3b82f6; }

        .bottom-nav { position: fixed; bottom: 0; left: 0; right: 0; background: white; border-top: 1px solid #e5e7eb; display: flex; justify-content: space-around; padding: 10px 0; z-index: 40; }
        .nav-item { flex: 1; text-align: center; padding: 5px; color: #6b7280; font-size: 11px; cursor: pointer; }
        .nav-item.active { color: #3b82f6; }
        .nav-item i { font-size: 20px; display: block; margin-bottom: 3px; }

        .password-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.8); z-index: 100; display: none; align-items: center; justify-content: center; }
        .password-overlay.active { display: flex; }
    </style>
</head>
<body class="text-gray-800 antialiased min-h-screen pb-20">

    <!-- Password Modal -->
    <div id="passwordModal" class="password-overlay">
        <div class="bg-white p-6 rounded-2xl w-80 mx-4">
            <h3 class="text-lg font-bold mb-4 text-center">🔒 Enter Password</h3>
            <input type="password" id="passwordInput" class="w-full px-4 py-3 border border-gray-200 rounded-lg mb-4 text-center text-2xl tracking-widest" placeholder="****" maxlength="4">
            <button onclick="verifyPassword()" class="w-full bg-blue-600 text-white py-3 rounded-lg font-bold">Unlock</button>
            <button onclick="closePasswordModal()" class="w-full mt-2 text-gray-500 py-2">Cancel</button>
        </div>
    </div>

    <!-- Header -->
    <header class="sticky top-0 z-40 glass-panel shadow-sm border-b border-gray-200">
        <div class="max-w-3xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center gap-3">
                <div class="bg-blue-600 text-white p-2 rounded-lg shadow-lg">
                    <i class="fa-solid fa-book-open"></i>
                </div>
                <div>
                    <h1 class="font-bold text-lg text-gray-900" id="appTitle">Work Book</h1>
                    <p class="text-xs text-gray-500" id="currentDateDisplay">Loading...</p>
                </div>
            </div>
            <div class="flex gap-2 items-center">
                <div class="flex gap-1">
                    <button onclick="setLang('en')" class="lang-btn" id="lang-en">EN</button>
                    <button onclick="setLang('hi')" class="lang-btn" id="lang-hi">हि</button>
                    <button onclick="setLang('gu')" class="lang-btn" id="lang-gu">ગુ</button>
                </div>
                <button onclick="openSettings()" class="p-2 text-gray-500 hover:bg-gray-100 rounded-full">
                    <i class="fa-solid fa-gear"></i>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content Area -->
    <main class="flex-grow p-4 max-w-3xl mx-auto w-full" id="mainContainer">
        <!-- Workers View -->
        <div id="workersView">
            <div id="workerListContainer" class="space-y-4">
                <div class="text-center py-12 text-gray-400">
                    <i class="fa-solid fa-users-slash text-4xl mb-3 opacity-30"></i>
                    <p id="noWorkersText">No workers added yet.</p>
                </div>
            </div>
        </div>

        <!-- Income Expense View (Hidden by default) -->
        <div id="financeView" class="hidden">
            <div class="bg-white rounded-2xl p-4 mb-4 shadow-sm">
                <h2 class="font-bold text-lg mb-4" id="financeTitle">Income & Expense</h2>
                
                <!-- Add Transaction -->
                <div class="grid grid-cols-2 gap-3 mb-4">
                    <button onclick="openFinanceModal('income')" class="bg-emerald-100 text-emerald-700 py-3 rounded-xl font-bold">
                        <i class="fa-solid fa-plus mr-2"></i><span id="addIncomeBtn">Add Income</span>
                    </button>
                    <button onclick="openFinanceModal('expense')" class="bg-red-100 text-red-700 py-3 rounded-xl font-bold">
                        <i class="fa-solid fa-minus mr-2"></i><span id="addExpenseBtn">Add Expense</span>
                    </button>
                </div>

                <!-- Filter by Place -->
                <select id="financePlaceFilter" onchange="renderFinance()" class="w-full p-3 border border-gray-200 rounded-lg mb-4 outline-none">
                    <option value="">All Places</option>
                </select>

                <!-- Summary -->
                <div class="grid grid-cols-3 gap-3 mb-4">
                    <div class="bg-emerald-50 p-3 rounded-xl text-center">
                        <div class="text-xs text-emerald-600" id="totalIncomeLabel">Income</div>
                        <div class="font-bold text-emerald-700" id="totalIncome">₹0</div>
                    </div>
                    <div class="bg-red-50 p-3 rounded-xl text-center">
                        <div class="text-xs text-red-600" id="totalExpenseLabel">Expense</div>
                        <div class="font-bold text-red-700" id="totalExpense">₹0</div>
                    </div>
                    <div class="bg-blue-50 p-3 rounded-xl text-center">
                        <div class="text-xs text-blue-600" id="balanceLabel">Balance</div>
                        <div class="font-bold text-blue-700" id="financeBalance">₹0</div>
                    </div>
                </div>

                <!-- List -->
                <div id="financeList" class="space-y-2"></div>
            </div>
        </div>
    </main>

    <!-- Floating Action Button (Only in Workers View) -->
    <div class="fixed bottom-20 right-6 z-30" id="fabButton">
        <button onclick="openAddWorkerModal()" class="bg-blue-600 hover:bg-blue-700 text-white w-14 h-14 rounded-full shadow-xl flex items-center justify-center text-2xl transition-transform hover:scale-110">
            <i class="fa-solid fa-plus"></i>
        </button>
    </div>

    <!-- Bottom Navigation -->
    <div class="bottom-nav">
        <div class="nav-item active" onclick="showView('workers')" id="navWorkers">
            <i class="fa-solid fa-users"></i>
            <span id="navWorkersText">Workers</span>
        </div>
        <div class="nav-item" onclick="showView('finance')" id="navFinance">
            <i class="fa-solid fa-wallet"></i>
            <span id="navFinanceText">Income/Exp</span>
        </div>
    </div>

    <!-- Add Worker Modal -->
    <div id="addWorkerModal" class="fixed inset-0 z-50 hidden">
        <div class="absolute inset-0 bg-black/40 backdrop-blur-sm" onclick="closeAddWorkerModal()"></div>
        <div class="absolute bottom-0 md:top-1/2 md:left-1/2 md:-translate-x-1/2 md:-translate-y-1/2 w-full md:w-[400px] bg-white md:rounded-2xl rounded-t-2xl p-6 shadow-2xl transform transition-transform duration-300 translate-y-full" id="addWorkerContent">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-bold" id="addWorkerTitle">Add New Worker</h2>
                <button onclick="closeAddWorkerModal()" class="text-gray-400"><i class="fa-solid fa-xmark text-xl"></i></button>
            </div>
            <div class="space-y-4">
                <input type="text" id="newWorkerName" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 outline-none" placeholder="Full Name">
                <div class="grid grid-cols-2 gap-4">
                    <input type="number" id="newWorkerWage" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 outline-none" placeholder="Daily Wage" value="500">
                    <input type="tel" id="newWorkerPhone" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 outline-none" placeholder="WhatsApp">
                </div>
                <button onclick="saveNewWorker()" class="w-full bg-blue-600 text-white font-semibold py-3 rounded-lg shadow-lg" id="saveWorkerBtn">Save Worker</button>
            </div>
        </div>
    </div>

    <!-- Quick Add Session Modal -->
    <div id="quickAddModal" class="fixed inset-0 z-50 hidden">
        <div class="absolute inset-0 bg-black/40 backdrop-blur-sm" onclick="closeQuickAddModal()"></div>
        <div class="absolute bottom-0 md:top-1/2 md:left-1/2 md:-translate-x-1/2 md:-translate-y-1/2 w-full md:w-[400px] bg-white md:rounded-2xl rounded-t-2xl p-6 shadow-2xl transform transition-transform duration-300 translate-y-full" id="quickAddContent">
            <div class="flex justify-between items-center mb-4">
                <div>
                    <h2 class="text-lg font-bold" id="quickAddWorkerName">Worker</h2>
                    <p class="text-xs text-gray-500" id="quickAddSubtitle">Add Session</p>
                </div>
                <button onclick="closeQuickAddModal()" class="text-gray-400"><i class="fa-solid fa-xmark text-xl"></i></button>
            </div>

            <div class="mb-4">
                <label class="block text-xs font-semibold text-gray-500 uppercase mb-2" id="dateLabel">Date</label>
                <input type="date" id="quickAddDate" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 outline-none">
            </div>

            <div id="quickAddWorkplaceSection" class="mb-4 hidden">
                <label class="block text-xs font-semibold text-gray-500 uppercase mb-2" id="placeLabel">Place</label>
                <div class="flex flex-wrap gap-2" id="quickAddWorkplaceList"></div>
            </div>

            <label class="block text-xs font-semibold text-gray-500 uppercase mb-2" id="typeLabel">Type</label>
            <div class="grid grid-cols-3 gap-3 mb-6">
                <label class="cursor-pointer">
                    <input type="radio" name="attendanceType" value="1" class="custom-radio sr-only" checked onchange="updateQuickAddPreview(1)">
                    <div class="border-2 border-gray-100 rounded-xl p-3 text-center hover:bg-gray-50">
                        <i class="fa-solid fa-sun text-amber-500"></i>
                        <div class="text-sm font-bold" id="fullDayLabel">Full</div>
                    </div>
                </label>
                <label class="cursor-pointer">
                    <input type="radio" name="attendanceType" value="0.5" class="custom-radio sr-only" onchange="updateQuickAddPreview(0.5)">
                    <div class="border-2 border-gray-100 rounded-xl p-3 text-center hover:bg-gray-50">
                        <i class="fa-solid fa-cloud-sun text-orange-400"></i>
                        <div class="text-sm font-bold" id="halfDayLabel">Half</div>
                    </div>
                </label>
                <label class="cursor-pointer">
                    <input type="radio" name="attendanceType" value="0" class="custom-radio sr-only" onchange="updateQuickAddPreview(0)">
                    <div class="border-2 border-gray-100 rounded-xl p-3 text-center hover:bg-gray-50">
                        <i class="fa-solid fa-moon text-indigo-400"></i>
                        <div class="text-sm font-bold" id="absentLabel">Absent</div>
                    </div>
                </label>
            </div>

            <div class="bg-gray-50 rounded-lg p-3 mb-4 flex justify-between">
                <span class="text-gray-500" id="earningLabel">Earning:</span>
                <span class="font-bold text-gray-800" id="quickAddEarningPreview">₹0</span>
            </div>

            <button onclick="confirmQuickAdd()" class="w-full bg-emerald-500 text-white font-semibold py-3 rounded-xl shadow-lg flex justify-center items-center gap-2" id="confirmAddBtn">
                <i class="fa-solid fa-check"></i> <span id="addSessionBtnText">Add Session</span>
            </button>
        </div>
    </div>

    <!-- Payment Modal -->
    <div id="payModal" class="fixed inset-0 z-50 hidden">
        <div class="absolute inset-0 bg-black/40 backdrop-blur-sm" onclick="closePayModal()"></div>
        <div class="absolute bottom-0 md:top-1/2 md:left-1/2 md:-translate-x-1/2 md:-translate-y-1/2 w-full md:w-[350px] bg-white md:rounded-2xl rounded-t-2xl p-6 shadow-2xl transform transition-transform duration-300 translate-y-full" id="payContent">
            <h3 class="text-lg font-bold mb-4" id="paymentTitle">Add Payment</h3>
            <input type="date" id="payDate" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-3 outline-none">
            <input type="number" id="payAmount" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-4 outline-none" placeholder="Amount (₹)">
            <button onclick="confirmPayment()" class="w-full bg-blue-600 text-white font-semibold py-3 rounded-lg" id="confirmPayBtn">Add Payment</button>
        </div>
    </div>

    <!-- WhatsApp Options Modal -->
    <div id="whatsappModal" class="fixed inset-0 z-50 hidden">
        <div class="absolute inset-0 bg-black/40 backdrop-blur-sm" onclick="closeWhatsappModal()"></div>
        <div class="absolute bottom-0 md:top-1/2 md:left-1/2 md:-translate-x-1/2 md:-translate-y-1/2 w-full md:w-[350px] bg-white md:rounded-2xl rounded-t-2xl p-6 shadow-2xl">
            <h3 class="text-lg font-bold mb-4">📱 Share Report</h3>
            <div class="space-y-3">
                <button onclick="shareTodayReport()" class="w-full p-4 bg-green-50 border border-green-200 rounded-xl text-left hover:bg-green-100">
                    <div class="font-bold text-green-800" id="todayReportBtn">Today's Report</div>
                    <div class="text-sm text-green-600" id="todayReportDesc">Quick summary of today</div>
                </button>
                <button onclick="shareFullReport()" class="w-full p-4 bg-blue-50 border border-blue-200 rounded-xl text-left hover:bg-blue-100">
                    <div class="font-bold text-blue-800" id="fullReportBtn">Full History</div>
                    <div class="text-sm text-blue-600" id="fullReportDesc">Complete work history</div>
                </button>
                <button onclick="generatePDF()" class="w-full p-4 bg-purple-50 border border-purple-200 rounded-xl text-left hover:bg-purple-100">
                    <div class="font-bold text-purple-800">Download PDF</div>
                    <div class="text-sm text-purple-600">Generate PDF report</div>
                </button>
            </div>
            <button onclick="closeWhatsappModal()" class="w-full mt-4 py-3 text-gray-500" id="cancelBtn">Cancel</button>
        </div>
    </div>

    <!-- Edit Session Modal -->
    <div id="editSessionModal" class="fixed inset-0 z-50 hidden">
        <div class="absolute inset-0 bg-black/40 backdrop-blur-sm" onclick="closeEditSessionModal()"></div>
        <div class="absolute bottom-0 md:top-1/2 md:left-1/2 md:-translate-x-1/2 md:-translate-y-1/2 w-full md:w-[350px] bg-white md:rounded-2xl rounded-t-2xl p-6 shadow-2xl">
            <h3 class="text-lg font-bold mb-4" id="editSessionTitle">Edit Session</h3>
            <input type="hidden" id="editSessionWorkerId">
            <input type="hidden" id="editSessionIdx">
            <input type="date" id="editSessionDate" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-3 outline-none">
            <select id="editSessionType" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-3 outline-none">
                <option value="1" id="optFull">Full Day (1.0)</option>
                <option value="0.5" id="optHalf">Half Day (0.5)</option>
                <option value="0" id="optAbsent">Absent (0)</option>
            </select>
            <input type="text" id="editSessionWorkplace" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-4 outline-none" placeholder="Place">
            <div class="flex gap-2">
                <button onclick="saveEditSession()" class="flex-1 bg-blue-600 text-white font-semibold py-3 rounded-lg" id="saveEditBtn">Save</button>
                <button onclick="requestPassword('deleteSession')" class="flex-1 bg-red-500 text-white font-semibold py-3 rounded-lg" id="deleteEditBtn">Delete</button>
            </div>
        </div>
    </div>

    <!-- Edit Payment Modal -->
    <div id="editPaymentModal" class="fixed inset-0 z-50 hidden">
        <div class="absolute inset-0 bg-black/40 backdrop-blur-sm" onclick="closeEditPaymentModal()"></div>
        <div class="absolute bottom-0 md:top-1/2 md:left-1/2 md:-translate-x-1/2 md:-translate-y-1/2 w-full md:w-[350px] bg-white md:rounded-2xl rounded-t-2xl p-6 shadow-2xl">
            <h3 class="text-lg font-bold mb-4" id="editPaymentTitle">Edit Payment</h3>
            <input type="hidden" id="editPaymentWorkerId">
            <input type="hidden" id="editPaymentIdx">
            <input type="date" id="editPaymentDate" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-3 outline-none">
            <input type="number" id="editPaymentAmount" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-4 outline-none" placeholder="Amount">
            <div class="flex gap-2">
                <button onclick="saveEditPayment()" class="flex-1 bg-blue-600 text-white font-semibold py-3 rounded-lg">Save</button>
                <button onclick="requestPassword('deletePayment')" class="flex-1 bg-red-500 text-white font-semibold py-3 rounded-lg">Delete</button>
            </div>
        </div>
    </div>

    <!-- Finance Modal -->
    <div id="financeModal" class="fixed inset-0 z-50 hidden">
        <div class="absolute inset-0 bg-black/40 backdrop-blur-sm" onclick="closeFinanceModal()"></div>
        <div class="absolute bottom-0 md:top-1/2 md:left-1/2 md:-translate-x-1/2 md:-translate-y-1/2 w-full md:w-[350px] bg-white md:rounded-2xl rounded-t-2xl p-6 shadow-2xl transform transition-transform duration-300 translate-y-full" id="financeModalContent">
            <h3 class="text-lg font-bold mb-4" id="financeModalTitle">Add Transaction</h3>
            <input type="date" id="financeDate" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-3 outline-none">
            <select id="financePlace" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-3 outline-none">
                <option value="">Select Place</option>
            </select>
            <input type="text" id="financeDesc" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-3 outline-none" placeholder="Description">
            <input type="number" id="financeAmount" class="w-full px-4 py-3 rounded-lg bg-gray-50 border border-gray-200 mb-4 outline-none" placeholder="Amount (₹)">
            <button onclick="saveFinance()" class="w-full bg-blue-600 text-white font-semibold py-3 rounded-lg" id="saveFinanceBtn">Save</button>
        </div>
    </div>

    <!-- Settings Modal -->
    <div id="settingsModal" class="fixed inset-0 z-50 hidden overflow-y-auto">
        <div class="absolute inset-0 bg-black/40 backdrop-blur-sm" onclick="closeSettings()"></div>
        <div class="relative bg-white min-h-full md:min-h-0 md:w-[500px] md:my-8 md:mx-auto md:rounded-2xl shadow-2xl p-6">
            <div class="flex justify-between items-center mb-6 border-b border-gray-100 pb-4">
                <h2 class="text-xl font-bold" id="settingsTitle">Settings</h2>
                <button onclick="closeSettings()" class="text-gray-400"><i class="fa-solid fa-xmark text-xl"></i></button>
            </div>

            <!-- Worker Edit Section -->
            <div class="mb-6">
                <h3 class="text-sm font-bold text-gray-400 uppercase mb-3" id="editWorkersTitle">Edit Workers</h3>
                <div id="editWorkerList" class="space-y-3"></div>
            </div>

            <div class="mb-6">
                <h3 class="text-sm font-bold text-gray-400 uppercase mb-3" id="placesTitle">Places</h3>
                <div class="flex gap-2 mb-3">
                    <input type="text" id="newWorkplaceInput" class="flex-1 px-3 py-2 border border-gray-200 rounded-lg text-sm outline-none" placeholder="Add place">
                    <button onclick="addWorkplace()" class="bg-gray-800 text-white px-4 py-2 rounded-lg text-sm" id="addPlaceBtn">Add</button>
                </div>
                <div class="flex flex-wrap gap-2" id="settingsWorkplaceList"></div>
            </div>

            <div>
                <h3 class="text-sm font-bold text-gray-400 uppercase mb-3" id="dangerZoneTitle">Danger Zone</h3>
                <div id="dangerWorkerList" class="space-y-2"></div>
            </div>
        </div>
    </div>

    <!-- Backup Modal -->
    <div id="backupModal" class="fixed inset-0 z-50 hidden">
        <div class="absolute inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center p-4">
            <div class="bg-white w-full max-w-md rounded-2xl shadow-2xl p-6">
                <h2 class="text-xl font-bold mb-2">Backup Data</h2>
                <textarea id="backupDataArea" class="w-full h-32 p-3 bg-gray-50 border border-gray-200 rounded-lg text-xs font-mono mb-4 outline-none resize-none"></textarea>
                <div class="grid grid-cols-2 gap-3">
                    <button onclick="copyBackupData()" class="bg-blue-100 text-blue-700 py-2 rounded-lg font-semibold">Copy</button>
                    <button onclick="restoreBackupData()" class="bg-emerald-100 text-emerald-700 py-2 rounded-lg font-semibold">Restore</button>
                </div>
                <button onclick="closeBackupModal()" class="w-full mt-3 text-gray-500 text-sm">Close</button>
            </div>
        </div>
    </div>

    <script>
        const DEFAULT_PASSWORD = "1234";
        let workers = [];
        let workplaces = [];
        let finances = [];
        let currentLang = localStorage.getItem('lang') || 'en';
        let currentQuickAddId = null;
        let selectedWorkplace = "";
        let currentPayWorkerId = null;
        let currentWhatsappWorkerId = null;
        let passwordCallback = null;
        let currentFinanceType = 'income';
        let currentEditWorker = null;
        let currentEditIdx = null;

        const translations = {
            en: {
                appTitle: "Work Book", addWorker: "Add New Worker", saveWorker: "Save Worker",
                fullDay: "Full", halfDay: "Half", absent: "Absent", date: "Date",
                place: "Place", type: "Type", earning: "Earning", addSession: "Add Session",
                payment: "Add Payment", todayReport: "Today's Report", fullReport: "Full History",
                cancel: "Cancel", workers: "Workers", incomeExpense: "Income/Exp",
                addIncome: "Add Income", addExpense: "Add Expense", income: "Income",
                expense: "Expense", balance: "Balance", settings: "Settings",
                editWorkers: "Edit Workers", places: "Places", dangerZone: "Danger Zone",
                addPlace: "Add", noWorkers: "No workers added yet.",
                editSession: "Edit Session", editPayment: "Edit Payment",
                save: "Save", delete: "Delete", settled: "SETTLED",
                summary: "Summary", byPlace: "By Place", byDate: "By Date",
                from: "From", to: "To", showSummary: "Show Summary"
            },
            hi: {
                appTitle: "वर्क बुक", addWorker: "नया कामगार", saveWorker: "सहेजें",
                fullDay: "पूरा", halfDay: "आधा", absent: "अनुपस्थित", date: "तारीख",
                place: "स्थान", type: "प्रकार", earning: "कमाई", addSession: "सत्र जोड़ें",
                payment: "भुगतान जोड़ें", todayReport: "आज की रिपोर्ट", fullReport: "पूरी रिपोर्ट",
                cancel: "रद्द", workers: "कामगार", incomeExpense: "आय/व्यय",
                addIncome: "आय जोड़ें", addExpense: "व्यय जोड़ें", income: "आय",
                expense: "व्यय", balance: "शेष", settings: "सेटिंग्स",
                editWorkers: "कामगार संपादित करें", places: "स्थान", dangerZone: "खतरा क्षेत्र",
                addPlace: "जोड़ें", noWorkers: "कोई कामगार नहीं।",
                editSession: "सत्र संपादित करें", editPayment: "भुगतान संपादित करें",
                save: "सहेजें", delete: "हटाएं", settled: "चुकता",
                summary: "सारांश", byPlace: "स्थान अनुसार", byDate: "तारीख अनुसार",
                from: "से", to: "तक", showSummary: "सारांश देखें"
            },
            gu: {
                appTitle: "વર્ક બુક", addWorker: "નવો મજૂર", saveWorker: "સાચવો",
                fullDay: "આખો", halfDay: "અડધો", absent: "ગેરહાજર", date: "તારીખ",
                place: "સ્થળ", type: "પ્રકાર", earning: "કમાણી", addSession: "સત્ર ઉમેરો",
                payment: "ચુકવણી ઉમેરો", todayReport: "આજની રિપોર્ટ", fullReport: "પૂર્ણ રિપોર્ટ",
                cancel: "રદ", workers: "મજૂર", incomeExpense: "આવક/ખર્ચ",
                addIncome: "આવક ઉમેરો", addExpense: "ખર્ચ ઉમેરો", income: "આવક",
                expense: "ખર્ચ", balance: "બાકી", settings: "સેટિંગ્સ",
                editWorkers: "મજૂર ફેરફાર", places: "સ્થળો", dangerZone: "જોખમ વિસ્તાર",
                addPlace: "ઉમેરો", noWorkers: "કોઈ મજૂર નથી.",
                editSession: "સત્ર ફેરફાર", editPayment: "ચુકવણી ફેરફાર",
                save: "સાચવો", delete: "કાઢી નાખો", settled: "ચુક્તા",
                summary: "સારાંશ", byPlace: "સ્થળ પ્રમાણે", byDate: "તારીખ પ્રમાણે",
                from: "થી", to: "સુધી", showSummary: "સારાંશ જુઓ"
            }
        };

        function t(key) {
            return translations[currentLang][key] || translations['en'][key];
        }

        function setLang(lang) {
            currentLang = lang;
            localStorage.setItem('lang', lang);
            document.querySelectorAll('.lang-btn').forEach(btn => btn.classList.remove('active'));
            document.getElementById(`lang-${lang}`).classList.add('active');
            updateUI();
            render();
        }

        function updateUI() {
            document.getElementById('appTitle').textContent = t('appTitle');
            document.getElementById('addWorkerTitle').textContent = t('addWorker');
            document.getElementById('saveWorkerBtn').textContent = t('saveWorker');
            document.getElementById('fullDayLabel').textContent = t('fullDay');
            document.getElementById('halfDayLabel').textContent = t('halfDay');
            document.getElementById('absentLabel').textContent = t('absent');
            document.getElementById('dateLabel').textContent = t('date');
            document.getElementById('placeLabel').textContent = t('place');
            document.getElementById('typeLabel').textContent = t('type');
            document.getElementById('earningLabel').textContent = t('earning');
            document.getElementById('addSessionBtnText').textContent = t('addSession');
            document.getElementById('paymentTitle').textContent = t('payment');
            document.getElementById('confirmPayBtn').textContent = t('payment');
            document.getElementById('todayReportBtn').textContent = t('todayReport');
            document.getElementById('fullReportBtn').textContent = t('fullReport');
            document.getElementById('cancelBtn').textContent = t('cancel');
            document.getElementById('navWorkersText').textContent = t('workers');
            document.getElementById('navFinanceText').textContent = t('incomeExpense');
            document.getElementById('financeTitle').textContent = t('incomeExpense');
            document.getElementById('addIncomeBtn').textContent = t('addIncome');
            document.getElementById('addExpenseBtn').textContent = t('addExpense');
            document.getElementById('totalIncomeLabel').textContent = t('income');
            document.getElementById('totalExpenseLabel').textContent = t('expense');
            document.getElementById('balanceLabel').textContent = t('balance');
            document.getElementById('settingsTitle').textContent = t('settings');
            document.getElementById('editWorkersTitle').textContent = t('editWorkers');
            document.getElementById('placesTitle').textContent = t('places');
            document.getElementById('dangerZoneTitle').textContent = t('dangerZone');
            document.getElementById('addPlaceBtn').textContent = t('addPlace');
            document.getElementById('noWorkersText').textContent = t('noWorkers');
            document.getElementById('editSessionTitle').textContent = t('editSession');
            document.getElementById('editPaymentTitle').textContent = t('editPayment');
            document.getElementById('saveEditBtn').textContent = t('save');
            document.getElementById('deleteEditBtn').textContent = t('delete');
            document.getElementById('optFull').textContent = t('fullDay') + ' (1.0)';
            document.getElementById('optHalf').textContent = t('halfDay') + ' (0.5)';
            document.getElementById('optAbsent').textContent = t('absent') + ' (0)';
            document.getElementById('financeModalTitle').textContent = currentFinanceType === 'income' ? t('addIncome') : t('addExpense');
            document.getElementById('saveFinanceBtn').textContent = t('save');
        }

        document.addEventListener('DOMContentLoaded', () => {
            loadData();
            setLang(currentLang);
            updateDateDisplay();
            render();
        });

        function updateDateDisplay() {
            const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
            document.getElementById('currentDateDisplay').textContent = new Date().toLocaleDateString(currentLang === 'en' ? 'en-IN' : (currentLang === 'hi' ? 'hi-IN' : 'gu-IN'), options);
        }

        function showView(view) {
            document.getElementById('workersView').classList.add('hidden');
            document.getElementById('financeView').classList.add('hidden');
            document.getElementById('fabButton').classList.add('hidden');
            document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('active'));
            
            if (view === 'workers') {
                document.getElementById('workersView').classList.remove('hidden');
                document.getElementById('fabButton').classList.remove('hidden');
                document.getElementById('navWorkers').classList.add('active');
            } else {
                document.getElementById('financeView').classList.remove('hidden');
                document.getElementById('navFinance').classList.add('active');
                renderFinance();
            }
        }

        // Password - Only for Delete
        function requestPassword(callback) {
            passwordCallback = callback;
            document.getElementById('passwordModal').classList.add('active');
            document.getElementById('passwordInput').value = '';
            setTimeout(() => document.getElementById('passwordInput').focus(), 100);
        }

        function verifyPassword() {
            const input = document.getElementById('passwordInput').value;
            if (input === DEFAULT_PASSWORD) {
                closePasswordModal();
                if (passwordCallback === 'deleteWorker') deleteWorkerReal();
                else if (passwordCallback === 'deleteSession') deleteEditSession();
                else if (passwordCallback === 'deletePayment') deleteEditPayment();
            } else {
                alert("Wrong password! Use: 1234");
            }
        }

        function closePasswordModal() {
            document.getElementById('passwordModal').classList.remove('active');
        }

        function calculateStats(worker) {
            const totalDays = worker.sessions.reduce((sum, s) => sum + s.val, 0);
            const totalWage = Math.round(totalDays * worker.wage);
            const totalPaid = worker.withdrawals.reduce((sum, w) => sum + w.amount, 0);
            const balance = totalWage - totalPaid;
            const isSettled = balance <= 0 && totalDays > 0;
            return { totalDays, totalWage, totalPaid, balance, isSettled };
        }

        function getCombinedHistory(worker) {
            const history = [];
            worker.sessions.forEach((s, idx) => {
                history.push({ type: 'session', date: s.date, place: s.workplace || '-', value: s.val, payment: 0, idx });
            });
            worker.withdrawals.forEach((w, idx) => {
                history.push({ type: 'payment', date: w.date, place: '', value: 0, payment: w.amount, idx });
            });
            return history.sort((a, b) => new Date(b.date) - new Date(a.date));
        }

        // Get unique places for a worker
        function getWorkerPlaces(worker) {
            const places = [...new Set(worker.sessions.map(s => s.workplace).filter(p => p))];
            return places.length > 0 ? places : ['General'];
        }

        function render() {
            renderWorkerList();
        }

        function renderWorkerList() {
            const container = document.getElementById('workerListContainer');
            
            if (workers.length === 0) {
                container.innerHTML = `
                    <div class="text-center py-12 bg-white rounded-2xl border border-dashed border-gray-300">
                        <div class="bg-blue-50 w-16 h-16 rounded-full flex items-center justify-center mx-auto mb-4">
                            <i class="fa-solid fa-user-plus text-blue-500 text-2xl"></i>
                        </div>
                        <h3 class="text-gray-900 font-semibold">${t('noWorkers')}</h3>
                    </div>
                `;
                return;
            }

            container.innerHTML = workers.map(worker => {
                const stats = calculateStats(worker);
                const isOpen = worker.isOpen ? 'open' : '';
                const balanceClass = stats.balance > 0 ? 'text-amber-600 bg-amber-50' : (stats.balance < 0 ? 'text-red-600 bg-red-50' : 'text-emerald-600 bg-emerald-50');
                const balanceText = stats.balance > 0 ? `₹${stats.balance}` : (stats.balance < 0 ? `Over: ₹${Math.abs(stats.balance)}` : '₹0');
                
                const history = getCombinedHistory(worker);
                const places = getWorkerPlaces(worker);
                
                // Check if settled and show separately
                let settledHtml = '';
                let activeHistory = history;
                
                if (stats.isSettled) {
                    // Find last zero balance point
                    let runningDays = 0;
                    let runningPaid = 0;
                    let settledIndex = -1;
                    
                    for (let i = history.length - 1; i >= 0; i--) {
                        const h = history[i];
                        if (h.type === 'session') runningDays += h.value;
                        else runningPaid += h.payment;
                        
                        const earned = Math.round(runningDays * worker.wage);
                        if (earned <= runningPaid && settledIndex === -1) {
                            settledIndex = i;
                        }
                    }
                    
                    if (settledIndex > 0) {
                        settledHtml = `
                            <div class="bg-emerald-50 border border-emerald-200 rounded-lg p-3 mb-3 text-center">
                                <span class="settled-badge">${t('settled')}</span>
                                <p class="text-xs text-emerald-600 mt-1">Previous sessions cleared</p>
                            </div>
                        `;
                        activeHistory = history.slice(0, settledIndex);
                    }
                }

                const historyHtml = activeHistory.length > 0 
                    ? activeHistory.map((h, i) => {
                        const dateObj = new Date(h.date);
                        const day = dateObj.getDate();
                        const month = dateObj.toLocaleString('default', { month: 'short' });
                        
                        if (h.type === 'session') {
                            return `
                                <div class="history-row cursor-pointer" onclick="openEditSession(${worker.id}, ${h.idx})">
                                    <div class="text-xs font-bold text-gray-600">${day} ${month}</div>
                                    <div class="text-sm text-gray-700 truncate">${h.place}</div>
                                    <div class="text-xs font-bold ${h.value===1?'text-emerald-600':(h.value===0.5?'text-amber-600':'text-gray-400')}">
                                        ${h.value===1?'1.0':(h.value===0.5?'0.5':'0')}
                                    </div>
                                    <div class="text-xs text-gray-400">-</div>
                                </div>
                            `;
                        } else {
                            return `
                                <div class="history-row cursor-pointer" onclick="openEditPayment(${worker.id}, ${h.idx})">
                                    <div class="text-xs font-bold text-gray-600">${day} ${month}</div>
                                    <div class="text-sm text-gray-500 italic">Payment</div>
                                    <div class="text-xs text-gray-400">-</div>
                                    <div class="text-xs font-bold text-red-600">₹${h.payment}</div>
                                </div>
                            `;
                        }
                    }).join('')
                    : `<div class="text-center py-4 text-gray-400 text-sm">No history</div>`;

                // Summary section with filters
                const summaryHtml = `
                    <div class="summary-box">
                        <div class="flex justify-between items-center mb-3">
                            <h4 class="font-bold text-gray-700">${t('summary')}</h4>
                            <span class="text-xs text-gray-500">${stats.totalDays} days</span>
                        </div>
                        
                        <!-- Filter Tabs -->
                        <div class="flex gap-2 mb-3">
                            <button onclick="toggleSummaryFilter(${worker.id}, 'place')" class="text-xs px-3 py-1 bg-blue-100 text-blue-700 rounded-full" id="filter-place-${worker.id}">${t('byPlace')}</button>
                            <button onclick="toggleSummaryFilter(${worker.id}, 'date')" class="text-xs px-3 py-1 bg-gray-100 text-gray-600 rounded-full" id="filter-date-${worker.id}">${t('byDate')}</button>
                        </div>

                        <!-- Place-wise Summary -->
                        <div id="summary-place-${worker.id}" class="space-y-2">
                            ${places.map(place => {
                                const placeSessions = worker.sessions.filter(s => (s.workplace || '-') === place);
                                const placeDays = placeSessions.reduce((sum, s) => sum + s.val, 0);
                                const placeEarn = Math.round(placeDays * worker.wage);
                                return `
                                    <div class="flex justify-between text-sm p-2 bg-gray-50 rounded">
                                        <span class="text-gray-600">${place}</span>
                                        <span class="font-bold text-gray-800">${placeDays}d = ₹${placeEarn}</span>
                                    </div>
                                `;
                            }).join('')}
                        </div>

                        <!-- Date-wise Summary (Hidden by default) -->
                        <div id="summary-date-${worker.id}" class="hidden space-y-2">
                            <div class="flex gap-2 mb-2">
                                <input type="date" id="date-from-${worker.id}" class="flex-1 text-xs p-2 border rounded" value="${new Date(Date.now() - 30*86400000).toISOString().split('T')[0]}">
                                <input type="date" id="date-to-${worker.id}" class="flex-1 text-xs p-2 border rounded" value="${new Date().toISOString().split('T')[0]}">
                            </div>
                            <button onclick="showDateSummary(${worker.id})" class="w-full bg-blue-600 text-white text-xs py-2 rounded">${t('showSummary')}</button>
                            <div id="date-summary-result-${worker.id}"></div>
                        </div>
                    </div>
                `;

                return `
                <div id="worker-card-${worker.id}" class="bg-white rounded-2xl shadow-sm border border-gray-100 overflow-hidden card-hover">
                    <div class="p-4 cursor-pointer flex justify-between items-center hover:bg-gray-50" onclick="toggleWorker(${worker.id})">
                        <div class="flex items-center gap-4">
                            <div class="w-10 h-10 rounded-full bg-gradient-to-br from-blue-100 to-blue-50 text-blue-600 flex items-center justify-center font-bold text-lg">
                                ${worker.name.charAt(0).toUpperCase()}
                            </div>
                            <div>
                                <h3 class="font-bold text-gray-800">${worker.name}</h3>
                                <div class="flex items-center gap-2 text-xs mt-0.5">
                                    <span class="text-gray-500">₹${worker.wage}/day</span>
                                    <span class="${balanceClass} px-2 py-0.5 rounded-full font-medium">${balanceText}</span>
                                    ${stats.isSettled ? '<span class="settled-badge text-[10px] py-0.5 px-2">' + t('settled') + '</span>' : ''}
                                </div>
                            </div>
                        </div>
                        <div class="flex items-center gap-2">
                            <button onclick="event.stopPropagation(); openQuickAdd(${worker.id})" class="w-8 h-8 rounded-full bg-emerald-100 text-emerald-600 flex items-center justify-center">
                                <i class="fa-solid fa-plus"></i>
                            </button>
                            ${worker.phone ? `
                            <button onclick="event.stopPropagation(); openWhatsappOptions(${worker.id})" class="w-8 h-8 rounded-full bg-green-100 text-green-600 flex items-center justify-center">
                                <i class="fa-brands fa-whatsapp"></i>
                            </button>` : ''}
                            <i class="fa-solid fa-chevron-down text-gray-400 transition-transform ${worker.isOpen ? 'rotate-180' : ''}"></i>
                        </div>
                    </div>

                    <div class="slide-down ${isOpen}" id="details-${worker.id}">
                        <div class="p-4 bg-gray-50 border-t border-gray-100">
                            ${settledHtml}
                            
                            <div class="grid grid-cols-4 gap-2 mb-4 text-center text-xs">
                                <div class="bg-white p-2 rounded-lg border">
                                    <div class="text-gray-400">${t('fullDay')}</div>
                                    <div class="font-bold text-gray-800">${stats.totalDays}</div>
                                </div>
                                <div class="bg-white p-2 rounded-lg border">
                                    <div class="text-gray-400">${t('earning')}</div>
                                    <div class="font-bold text-emerald-600">₹${stats.totalWage}</div>
                                </div>
                                <div class="bg-white p-2 rounded-lg border">
                                    <div class="text-gray-400">${t('payment')}</div>
                                    <div class="font-bold text-blue-600">₹${stats.totalPaid}</div>
                                </div>
                                <div class="bg-white p-2 rounded-lg border">
                                    <div class="text-gray-400">${t('balance')}</div>
                                    <div class="font-bold ${stats.balance >= 0 ? 'text-amber-600' : 'text-red-600'}">₹${stats.balance}</div>
                                </div>
                            </div>

                            <button onclick="openPayModal(${worker.id})" class="w-full mb-4 bg-blue-600 text-white py-2 rounded-lg font-semibold flex items-center justify-center gap-2">
                                <i class="fa-solid fa-money-bill-wave"></i> ${t('payment')}
                            </button>

                            <div class="bg-white rounded-xl border border-gray-200 overflow-hidden mb-3">
                                <div class="grid grid-cols-4 gap-2 p-3 bg-gray-100 text-xs font-bold text-gray-500 uppercase text-center">
                                    <div>${t('date')}</div>
                                    <div>${t('place')}</div>
                                    <div>Day</div>
                                    <div>₹</div>
                                </div>
                                <div class="max-h-60 overflow-y-auto">
                                    ${historyHtml}
                                </div>
                            </div>

                            ${summaryHtml}
                        </div>
                    </div>
                </div>
                `;
            }).join('');
        }

        function toggleSummaryFilter(workerId, type) {
            document.getElementById(`summary-place-${workerId}`).classList.add('hidden');
            document.getElementById(`summary-date-${workerId}`).classList.add('hidden');
            document.getElementById(`filter-place-${workerId}`).className = 'text-xs px-3 py-1 bg-gray-100 text-gray-600 rounded-full';
            document.getElementById(`filter-date-${workerId}`).className = 'text-xs px-3 py-1 bg-gray-100 text-gray-600 rounded-full';
            
            document.getElementById(`summary-${type}-${workerId}`).classList.remove('hidden');
            document.getElementById(`filter-${type}-${workerId}`).className = 'text-xs px-3 py-1 bg-blue-100 text-blue-700 rounded-full';
        }

        function showDateSummary(workerId) {
            const worker = workers.find(w => w.id === workerId);
            const from = document.getElementById(`date-from-${workerId}`).value;
            const to = document.getElementById(`date-to-${workerId}`).value;
            
            const filtered = worker.sessions.filter(s => s.date >= from && s.date <= to);
            const days = filtered.reduce((sum, s) => sum + s.val, 0);
            const earn = Math.round(days * worker.wage);
            
            document.getElementById(`date-summary-result-${workerId}`).innerHTML = `
                <div class="bg-blue-50 p-3 rounded-lg text-center mt-2">
                    <div class="text-xs text-blue-600">${from} ${t('to')} ${to}</div>
                    <div class="font-bold text-blue-800 text-lg">${days} days = ₹${earn}</div>
                </div>
            `;
        }

        function toggleWorker(id) {
            const worker = workers.find(w => w.id === id);
            if (worker) {
                worker.isOpen = !worker.isOpen;
                const details = document.getElementById(`details-${id}`);
                const icon = details.previousElementSibling.querySelector('.fa-chevron-down');
                
                if(worker.isOpen) {
                    details.classList.add('open');
                    icon.classList.add('rotate-180');
                } else {
                    details.classList.remove('open');
                    icon.classList.remove('rotate-180');
                }
            }
        }

        // Payment Modal
        function openPayModal(id) {
            currentPayWorkerId = id;
            document.getElementById('payDate').value = new Date().toISOString().split('T')[0];
            document.getElementById('payAmount').value = '';
            document.getElementById('payModal').classList.remove('hidden');
            setTimeout(() => {
                document.getElementById('payContent').classList.remove('translate-y-full');
            }, 10);
        }

        function closePayModal() {
            document.getElementById('payContent').classList.add('translate-y-full');
            setTimeout(() => document.getElementById('payModal').classList.add('hidden'), 300);
        }

        function confirmPayment() {
            const amount = parseFloat(document.getElementById('payAmount').value);
            const date = document.getElementById('payDate').value;
            
            if (!amount || amount <= 0 || !date) return alert('Enter valid amount and date');
            
            const worker = workers.find(w => w.id === currentPayWorkerId);
            worker.withdrawals.push({ date, amount });
            worker.withdrawals.sort((a, b) => new Date(a.date) - new Date(b.date));
            
            saveData();
            render();
            closePayModal();
            
            setTimeout(() => {
                const w = workers.find(x => x.id === currentPayWorkerId);
                w.isOpen = true;
                renderWorkerList();
                const details = document.getElementById(`details-${currentPayWorkerId}`);
                const icon = details.previousElementSibling.querySelector('.fa-chevron-down');
                details.classList.add('open');
                icon.classList.add('rotate-180');
            }, 50);
        }

        // Edit Session/Payment - No password for edit, only for delete
        function openEditSession(workerId, idx) {
            const worker = workers.find(w => w.id === workerId);
            const session = worker.sessions[idx];
            document.getElementById('editSessionWorkerId').value = workerId;
            document.getElementById('editSessionIdx').value = idx;
            document.getElementById('editSessionDate').value = session.date;
            document.getElementById('editSessionType').value = session.val;
            document.getElementById('editSessionWorkplace').value = session.workplace || '';
            document.getElementById('editSessionModal').classList.remove('hidden');
        }

        function openEditPayment(workerId, idx) {
            const worker = workers.find(w => w.id === workerId);
            const payment = worker.withdrawals[idx];
            document.getElementById('editPaymentWorkerId').value = workerId;
            document.getElementById('editPaymentIdx').value = idx;
            document.getElementById('editPaymentDate').value = payment.date;
            document.getElementById('editPaymentAmount').value = payment.amount;
            document.getElementById('editPaymentModal').classList.remove('hidden');
        }

        function saveEditSession() {
            const workerId = parseInt(document.getElementById('editSessionWorkerId').value);
            const idx = parseInt(document.getElementById('editSessionIdx').value);
            const worker = workers.find(w => w.id === workerId);
            
            worker.sessions[idx] = {
                date: document.getElementById('editSessionDate').value,
                val: parseFloat(document.getElementById('editSessionType').value),
                workplace: document.getElementById('editSessionWorkplace').value
            };
            worker.sessions.sort((a, b) => new Date(a.date) - new Date(b.date));
            saveData();
            render();
            closeEditSessionModal();
        }

        function deleteEditSession() {
            const workerId = parseInt(document.getElementById('editSessionWorkerId').value);
            const idx = parseInt(document.getElementById('editSessionIdx').value);
            const worker = workers.find(w => w.id === workerId);
            worker.sessions.splice(idx, 1);
            saveData();
            render();
            closeEditSessionModal();
        }

        function closeEditSessionModal() {
            document.getElementById('editSessionModal').classList.add('hidden');
        }

        function saveEditPayment() {
            const workerId = parseInt(document.getElementById('editPaymentWorkerId').value);
            const idx = parseInt(document.getElementById('editPaymentIdx').value);
            const worker = workers.find(w => w.id === workerId);
            
            worker.withdrawals[idx] = {
                date: document.getElementById('editPaymentDate').value,
                amount: parseFloat(document.getElementById('editPaymentAmount').value)
            };
            worker.withdrawals.sort((a, b) => new Date(a.date) - new Date(b.date));
            saveData();
            render();
            closeEditPaymentModal();
        }

        function deleteEditPayment() {
            const workerId = parseInt(document.getElementById('editPaymentWorkerId').value);
            const idx = parseInt(document.getElementById('editPaymentIdx').value);
            const worker = workers.find(w => w.id === workerId);
            worker.withdrawals.splice(idx, 1);
            saveData();
            render();
            closeEditPaymentModal();
        }

        function closeEditPaymentModal() {
            document.getElementById('editPaymentModal').classList.add('hidden');
        }

        // Finance/Income Expense
        function openFinanceModal(type) {
            currentFinanceType = type;
            document.getElementById('financeModalTitle').textContent = type === 'income' ? t('addIncome') : t('addExpense');
            document.getElementById('financeDate').value = new Date().toISOString().split('T')[0];
            document.getElementById('financeAmount').value = '';
            document.getElementById('financeDesc').value = '';
            
            const placeSelect = document.getElementById('financePlace');
            placeSelect.innerHTML = '<option value="">' + (currentLang === 'en' ? 'Select Place' : (currentLang === 'hi' ? 'स्थान चुनें' : 'સ્થળ પસંદ કરો')) + '</option>';
            workplaces.forEach(wp => {
                placeSelect.innerHTML += `<option value="${wp}">${wp}</option>`;
            });
            
            document.getElementById('financeModal').classList.remove('hidden');
            setTimeout(() => {
                document.getElementById('financeModalContent').classList.remove('translate-y-full');
            }, 10);
        }

        function closeFinanceModal() {
            document.getElementById('financeModalContent').classList.add('translate-y-full');
            setTimeout(() => document.getElementById('financeModal').classList.add('hidden'), 300);
        }

        function saveFinance() {
            const date = document.getElementById('financeDate').value;
            const place = document.getElementById('financePlace').value;
            const desc = document.getElementById('financeDesc').value;
            const amount = parseFloat(document.getElementById('financeAmount').value);
            
            if (!date || !amount) return alert('Fill all fields');
            
            finances.push({
                id: Date.now(),
                type: currentFinanceType,
                date,
                place: place || 'General',
                desc: desc || (currentFinanceType === 'income' ? 'Income' : 'Expense'),
                amount
            });
            
            saveData();
            renderFinance();
            closeFinanceModal();
        }

        function renderFinance() {
            const filterPlace = document.getElementById('financePlaceFilter').value;
            const filtered = filterPlace ? finances.filter(f => f.place === filterPlace) : finances;
            
            const income = filtered.filter(f => f.type === 'income').reduce((sum, f) => sum + f.amount, 0);
            const expense = filtered.filter(f => f.type === 'expense').reduce((sum, f) => sum + f.amount, 0);
            
            document.getElementById('totalIncome').textContent = '₹' + income;
            document.getElementById('totalExpense').textContent = '₹' + expense;
            document.getElementById('financeBalance').textContent = '₹' + (income - expense);
            
            // Update place filter options
            const allPlaces = [...new Set(finances.map(f => f.place))];
            const currentFilter = document.getElementById('financePlaceFilter').value;
            document.getElementById('financePlaceFilter').innerHTML = '<option value="">' + (currentLang === 'en' ? 'All Places' : (currentLang === 'hi' ? 'सभी स्थान' : 'બધા સ્થળો')) + '</option>' + 
                allPlaces.map(p => `<option value="${p}" ${p === currentFilter ? 'selected' : ''}>${p}</option>`).join('');
            
            const list = document.getElementById('financeList');
            const sorted = filtered.sort((a, b) => new Date(b.date) - new Date(a.date));
            
            list.innerHTML = sorted.map(f => `
                <div class="flex justify-between items-center p-3 bg-white rounded-lg border ${f.type === 'income' ? 'border-emerald-200 bg-emerald-50' : 'border-red-200 bg-red-50'}">
                    <div>
                        <div class="font-bold text-sm ${f.type === 'income' ? 'text-emerald-800' : 'text-red-800'}">${f.desc}</div>
                        <div class="text-xs text-gray-500">${f.date} • ${f.place}</div>
                    </div>
                    <div class="font-bold ${f.type === 'income' ? 'text-emerald-600' : 'text-red-600'}">
                        ${f.type === 'income' ? '+' : '-'}₹${f.amount}
                    </div>
                </div>
            `).join('');
        }

        // Other modals
        function openAddWorkerModal() {
            document.getElementById('addWorkerModal').classList.remove('hidden');
            setTimeout(() => {
                document.getElementById('addWorkerContent').classList.remove('translate-y-full', 'scale-95', 'opacity-0');
            }, 10);
        }

        function closeAddWorkerModal() {
            document.getElementById('addWorkerContent').classList.add('translate-y-full', 'scale-95', 'opacity-0');
            setTimeout(() => document.getElementById('addWorkerModal').classList.add('hidden'), 300);
        }

        function saveNewWorker() {
            const name = document.getElementById('newWorkerName').value.trim();
            const wage = parseFloat(document.getElementById('newWorkerWage').value) || 0;
            const phone = document.getElementById('newWorkerPhone').value.trim();

            if (!name) return alert("Enter name");

            workers.push({
                id: Date.now(),
                name,
                wage,
                phone: phone.replace(/\D/g, ''),
                sessions: [],
                withdrawals: [],
                isOpen: false
            });

            saveData();
            render();
            closeAddWorkerModal();
            document.getElementById('newWorkerName').value = '';
            document.getElementById('newWorkerPhone').value = '';
        }

        function openQuickAdd(id) {
            currentQuickAddId = id;
            const worker = workers.find(w => w.id === id);
            document.getElementById('quickAddWorkerName').textContent = worker.name;
            document.getElementById('quickAddDate').value = new Date().toISOString().split('T')[0];
            
            selectedWorkplace = worker.sessions.length > 0 ? worker.sessions[worker.sessions.length-1].workplace : (workplaces[0] || "");
            document.querySelector('input[name="attendanceType"][value="1"]').checked = true;
            updateQuickAddPreview(1);

            const wpContainer = document.getElementById('quickAddWorkplaceList');
            const wpSection = document.getElementById('quickAddWorkplaceSection');
            
            if (workplaces.length > 0) {
                wpSection.classList.remove('hidden');
                wpContainer.innerHTML = workplaces.map(wp => `
                    <button onclick="selectQuickAddWorkplace('${wp}')" id="wp-btn-${wp}" 
                        class="px-3 py-1 rounded-lg text-xs font-medium border ${selectedWorkplace === wp ? 'bg-blue-600 text-white border-blue-600' : 'bg-white text-gray-600 border-gray-200'}">
                        ${wp}
                    </button>
                `).join('');
            } else {
                wpSection.classList.add('hidden');
            }

            document.getElementById('quickAddModal').classList.remove('hidden');
            setTimeout(() => {
                document.getElementById('quickAddContent').classList.remove('translate-y-full', 'scale-95', 'opacity-0');
            }, 10);
        }

        function selectQuickAddWorkplace(wp) {
            selectedWorkplace = wp;
            workplaces.forEach(p => {
                const btn = document.getElementById(`wp-btn-${p}`);
                if (btn) {
                    btn.className = p === wp 
                        ? "px-3 py-1 rounded-lg text-xs font-medium border bg-blue-600 text-white border-blue-600"
                        : "px-3 py-1 rounded-lg text-xs font-medium border bg-white text-gray-600 border-gray-200";
                }
            });
        }

        function updateQuickAddPreview(val) {
            const worker = workers.find(w => w.id === currentQuickAddId);
            document.getElementById('quickAddEarningPreview').textContent = '₹' + Math.round(val * worker.wage);
        }

        function closeQuickAddModal() {
            document.getElementById('quickAddContent').classList.add('translate-y-full', 'scale-95', 'opacity-0');
            setTimeout(() => document.getElementById('quickAddModal').classList.add('hidden'), 300);
        }

        function confirmQuickAdd() {
            const type = parseFloat(document.querySelector('input[name="attendanceType"]:checked').value);
            const date = document.getElementById('quickAddDate').value;
            
            if (!date) return alert('Select date');

            const worker = workers.find(w => w.id === currentQuickAddId);

            if (worker.sessions.some(s => s.date === date)) {
                if(!confirm('Date exists. Overwrite?')) return;
                worker.sessions = worker.sessions.filter(s => s.date !== date);
            }

            worker.sessions.push({ date, val: type, workplace: selectedWorkplace });
            worker.sessions.sort((a, b) => new Date(a.date) - new Date(b.date));

            saveData();
            render();
            closeQuickAddModal();
        }

        function openWhatsappOptions(id) {
            currentWhatsappWorkerId = id;
            document.getElementById('whatsappModal').classList.remove('hidden');
        }

        function closeWhatsappModal() {
            document.getElementById('whatsappModal').classList.add('hidden');
        }

        function shareTodayReport() {
            const worker = workers.find(w => w.id === currentWhatsappWorkerId);
            const today = new Date().toISOString().split('T')[0];
            const todaySessions = worker.sessions.filter(s => s.date === today);
            const todayPay = worker.withdrawals.filter(w => w.date === today);
            
            const totalToday = todaySessions.reduce((sum, s) => sum + s.val, 0);
            const payToday = todayPay.reduce((sum, w) => sum + w.amount, 0);
            
            const msg = `*${worker.name} - ${t('todayReport')}*\n` +
                `📅 ${today}\n` +
                `⏰ ${totalToday} days\n` +
                `💰 ₹${Math.round(totalToday * worker.wage)}\n` +
                `💸 ₹${payToday}\n\n` +
                `_Work Book_`;
            
            window.open(`https://wa.me/${worker.phone}?text=${encodeURIComponent(msg)}`);
            closeWhatsappModal();
        }

        function shareFullReport() {
            const worker = workers.find(w => w.id === currentWhatsappWorkerId);
            const stats = calculateStats(worker);
            
            let msg = `*${worker.name} - ${t('fullReport')}*\n\n` +
                `📊 ${t('summary')}:\n` +
                `• ${t('fullDay')}: ${stats.totalDays}\n` +
                `• ${t('earning')}: ₹${stats.totalWage}\n` +
                `• ${t('payment')}: ₹${stats.totalPaid}\n` +
                `• ${t('balance')}: ₹${stats.balance}\n\n`;
            
            const history = getCombinedHistory(worker).slice(0, 10);
            history.forEach(h => {
                const date = new Date(h.date).toLocaleDateString();
                if (h.type === 'session') {
                    msg += `• ${date}: ${h.place} ${h.value===1?'F':(h.value===0.5?'H':'A')}\n`;
                } else {
                    msg += `• ${date}: Pay ₹${h.payment}\n`;
                }
            });
            
            msg += `\n_Work Book_`;
            window.open(`https://wa.me/${worker.phone}?text=${encodeURIComponent(msg)}`);
            closeWhatsappModal();
        }

        function generatePDF() {
            const { jsPDF } = window.jspdf;
            const worker = workers.find(w => w.id === currentWhatsappWorkerId);
            const stats = calculateStats(worker);
            
            const doc = new jsPDF();
            doc.setFontSize(20);
            doc.text(`${worker.name}`, 20, 30);
            
            doc.setFontSize(12);
            doc.text(`${new Date().toLocaleString()}`, 20, 45);
            
            doc.setFontSize(14);
            doc.text(t('summary'), 20, 65);
            doc.setFontSize(11);
            doc.text(`${t('fullDay')}: ${stats.totalDays}`, 20, 75);
            doc.text(`${t('earning')}: ₹${stats.totalWage}`, 20, 82);
            doc.text(`${t('payment')}: ₹${stats.totalPaid}`, 20, 89);
            doc.text(`${t('balance')}: ₹${stats.balance}`, 20, 96);
            
            let y = 120;
            const history = getCombinedHistory(worker);
            
            doc.setFontSize(9);
            doc.text(t('date'), 20, y);
            doc.text(t('place'), 70, y);
            doc.text('Type', 120, y);
            doc.text('Amt', 170, y);
            y += 7;
            
            history.forEach(h => {
                if (y > 280) { doc.addPage(); y = 30; }
                const date = new Date(h.date).toLocaleDateString();
                if (h.type === 'session') {
                    doc.text(date, 20, y);
                    doc.text(h.place.substring(0, 15), 70, y);
                    doc.text(h.value===1?'Full':(h.value===0.5?'Half':'Abs'), 120, y);
                    doc.text(`+₹${Math.round(h.value * worker.wage)}`, 170, y);
                } else {
                    doc.text(date, 20, y);
                    doc.text('Payment', 70, y);
                    doc.text('', 120, y);
                    doc.text(`-₹${h.payment}`, 170, y);
                }
                y += 6;
            });
            
            doc.save(`${worker.name}_Report.pdf`);
            closeWhatsappModal();
        }

        // Settings with Worker Edit
        function openSettings() {
            document.getElementById('settingsModal').classList.remove('hidden');
            renderSettings();
        }

        function closeSettings() {
            document.getElementById('settingsModal').classList.add('hidden');
        }

        function renderSettings() {
            // Edit Workers List
            document.getElementById('editWorkerList').innerHTML = workers.map(w => `
                <div class="bg-gray-50 p-3 rounded-lg border">
                    <div class="grid grid-cols-1 gap-2 mb-2">
                        <input type="text" value="${w.name}" onchange="updateWorkerName(${w.id}, this.value)" class="w-full px-2 py-1 border rounded text-sm" placeholder="Name">
                        <div class="grid grid-cols-2 gap-2">
                            <input type="number" value="${w.wage}" onchange="updateWorkerWage(${w.id}, this.value)" class="w-full px-2 py-1 border rounded text-sm" placeholder="Wage">
                            <input type="tel" value="${w.phone}" onchange="updateWorkerPhone(${w.id}, this.value)" class="w-full px-2 py-1 border rounded text-sm" placeholder="Phone">
                        </div>
                    </div>
                </div>
            `).join('');

            // Places
            document.getElementById('settingsWorkplaceList').innerHTML = workplaces.map((wp, idx) => `
                <div class="flex items-center justify-between bg-gray-50 px-3 py-2 rounded-lg border w-full">
                    <span class="text-sm font-medium">${wp}</span>
                    <button onclick="deleteWorkplace(${idx})" class="text-red-400"><i class="fa-solid fa-trash text-xs"></i></button>
                </div>
            `).join('');

            // Danger Zone - Delete Workers
            document.getElementById('dangerWorkerList').innerHTML = workers.map(w => `
                <div class="flex items-center justify-between p-3 bg-red-50 border border-red-100 rounded-lg">
                    <span class="font-medium text-red-800">${w.name}</span>
                    <button onclick="requestPassword('deleteWorker'); workerToDelete=${w.id}" class="px-3 py-1 bg-red-500 text-white rounded text-xs font-bold">Delete</button>
                </div>
            `).join('');
        }

        function updateWorkerName(id, name) {
            const w = workers.find(x => x.id === id);
            if (w && name.trim()) w.name = name.trim();
            saveData();
        }

        function updateWorkerWage(id, wage) {
            const w = workers.find(x => x.id === id);
            if (w) w.wage = parseFloat(wage) || 0;
            saveData();
        }

        function updateWorkerPhone(id, phone) {
            const w = workers.find(x => x.id === id);
            if (w) w.phone = phone.replace(/\D/g, '');
            saveData();
        }

        let workerToDelete = null;

        function deleteWorkerReal() {
            workers = workers.filter(w => w.id !== workerToDelete);
            saveData();
            render();
            renderSettings();
        }

        function addWorkplace() {
            const val = document.getElementById('newWorkplaceInput').value.trim();
            if (val && !workplaces.includes(val)) {
                workplaces.push(val);
                document.getElementById('newWorkplaceInput').value = '';
                saveData();
                renderSettings();
            }
        }

        function deleteWorkplace(idx) {
            workplaces.splice(idx, 1);
            saveData();
            renderSettings();
        }

        // Backup
        function openBackupModal() {
            const dataStr = JSON.stringify({workers, workplaces, finances});
            document.getElementById('backupDataArea').value = btoa(unescape(encodeURIComponent(dataStr)));
            document.getElementById('backupModal').classList.remove('hidden');
        }

        function closeBackupModal() {
            document.getElementById('backupModal').classList.add('hidden');
        }

        function copyBackupData() {
            document.getElementById("backupDataArea").select();
            document.execCommand("copy");
            event.target.textContent = "Copied!";
            setTimeout(() => event.target.textContent = "Copy", 2000);
        }

        function restoreBackupData() {
            try {
                const encoded = document.getElementById('backupDataArea').value.trim();
                const jsonStr = decodeURIComponent(escape(atob(encoded)));
                const data = JSON.parse(jsonStr);
                if (data.workers) workers = data.workers;
                if (data.workplaces) workplaces = data.workplaces;
                if (data.finances) finances = data.finances;
                saveData();
                render();
                closeBackupModal();
                alert("Restored!");
            } catch (e) {
                alert("Invalid code");
            }
        }

        function saveData() {
            localStorage.setItem('workbook_data', JSON.stringify({workers, workplaces, finances}));
        }

        function loadData() {
            const data = localStorage.getItem('workbook_data');
            if (data) {
                try {
                    const parsed = JSON.parse(data);
                    workers = parsed.workers || [];
                    workplaces = parsed.workplaces || [];
                    finances = parsed.finances || [];
                } catch(e) {}
            }
        }
    </script>
</body>
</html>
