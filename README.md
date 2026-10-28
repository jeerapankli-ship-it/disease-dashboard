# disease-dashboard
!pip install -q streamlit
!wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
!dpkg -i cloudflared-linux-amd64.deb

%%writefile index.html
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบสารสนเทศและเฝ้าระวังแนวโน้มโรคตามฤดูกาล (Epidemiological Surveillance Dashboard)</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js for Responsive Medical Data Charts -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Google Fonts - Sarabun -->
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Sarabun', 'sans-serif'],
                    },
                    colors: {
                        medical: {
                            50: '#F0F7FF',
                            100: '#E0EFFE',
                            200: '#BAE0FD',
                            500: '#0284C7',
                            600: '#0369A1',
                            700: '#0284C7',
                            800: '#075985',
                            900: '#0F172A',
                            accent: '#0052CC',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        body {
            font-family: 'Sarabun', sans-serif;
            background-color: #F8FAFC;
            color: #1E293B;
        }
        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: #F1F5F9;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: #CBD5E1;
            border-radius: 4px;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between custom-scrollbar">

    <!-- 1. POPUP EMERGENCY MODAL BANNER -->
    <div id="emergencyModal" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-900/75 p-4 transition-opacity duration-300">
        <div class="bg-white rounded-2xl shadow-2xl max-w-2xl w-full border-t-8 border-red-600 overflow-hidden transform transition-all scale-100">
            <div class="p-6 md:p-8">
                <div class="flex items-center space-x-3 text-red-600 mb-4">
                    <div class="p-3 bg-red-100 rounded-full">
                        <i data-lucide="siren" class="w-8 h-8 animate-pulse"></i>
                    </div>
                    <div>
                        <span class="text-xs font-bold tracking-wider uppercase text-red-500 bg-red-50 px-2.5 py-1 rounded-md border border-red-200">ประกาศเตือนภัยด่วนฉุกเฉิน</span>
                        <h3 class="text-xl md:text-2xl font-bold text-slate-900 mt-1">สถานการณ์เฝ้าระวังโรคระบาดประจำฤดูกาล</h3>
                    </div>
                </div>

                <div class="space-y-4 text-slate-600 text-sm md:text-base leading-relaxed bg-slate-50 p-4 rounded-xl border border-slate-200">
                    <p class="font-medium text-slate-800">
                        🚨 <span class="font-bold text-red-600">แจ้งเตือนระดับพิเศษ:</span> ขณะนี้สถิติการเจ็บป่วยด้วย <span class="font-bold text-slate-900">โรคไข้หวัดใหญ่</span> และ <span class="font-bold text-slate-900">โรคไข้เลือดออก</span> มีแนวโน้มพุ่งสูงขึ้นเกินค่าเฉลี่ยปกติในหลายพื้นที่
                    </p>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-3 text-xs md:text-sm pt-2">
                        <div class="bg-white p-3 rounded-lg border border-slate-200 shadow-sm">
                            <span class="font-bold text-blue-600 block mb-1">👥 สำหรับประชาชน</span>
                            สวมหน้ากากอนามัยในที่ชุมชน ล้างมือบ่อยๆ และทำลายแหล่งเพาะพันธุ์ยุงลายรอบบ้าน
                        </div>
                        <div class="bg-white p-3 rounded-lg border border-slate-200 shadow-sm">
                            <span class="font-bold text-emerald-600 block mb-1">👨‍⚕️ สำหรับสถานพยาบาล</span>
                            สำรองเวชภัณฑ์ ยาลดไข้ ยาต้านไวรัส และเตรียมเปิดใช้แผนรับมือโรคระบาด
                        </div>
                    </div>
                </div>

                <div class="mt-6 flex flex-col sm:flex-row items-center justify-between gap-3 pt-2">
                    <span class="text-xs text-slate-400">ศูนย์สารสนเทศและเฝ้าระวังโรคระบาด (DDC System)</span>
                    <button onclick="closeEmergencyModal()" class="w-full sm:w-auto bg-gradient-to-r from-blue-600 to-sky-600 hover:from-blue-700 hover:to-sky-700 text-white font-semibold px-6 py-3 rounded-xl shadow-lg shadow-blue-500/30 transition-all flex items-center justify-center space-x-2">
                        <span>รับทราบข้อความ & เข้าสู่ระบบสารสนเทศ</span>
                        <i data-lucide="arrow-right" class="w-4 h-4"></i>
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- 2. TOP OFFICIAL HEADER BAR (ปรับเป็นแถบสีฟ้าแล้ว) -->
    <header class="bg-slate-900 text-white shadow-md sticky top-0 z-30">
        <div class="bg-sky-800 text-sky-50 text-xs px-4 py-1.5 flex justify-between items-center border-b border-sky-700">
            <div class="flex items-center space-x-2">
                <i data-lucide="building-2" class="w-3.5 h-3.5"></i>
                <span class="font-medium">กระทรวงสาธารณสุข | Ministry of Public Health</span>
            </div>
            <div class="flex items-center space-x-4">
                <span class="font-bold bg-white/20 px-2 py-0.5 rounded text-white flex items-center gap-1">
                    <i data-lucide="phone-call" class="w-3 h-3"></i> สายด่วนกรมควบคุมโรค 1422
                </span>
            </div>
        </div>

        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3.5 flex items-center justify-between">
            <div class="flex items-center space-x-3">
                <div class="bg-white p-2 rounded-xl shadow-md">
                    <i data-lucide="activity" class="w-7 h-7 text-blue-600"></i>
                </div>
                <div>
                    <h1 class="text-lg md:text-xl font-extrabold tracking-tight text-white flex items-center gap-2">
                        ระบบสารสนเทศและเฝ้าระวังโรคระบาดตามฤดูกาล
                    </h1>
                    <p class="text-xs text-blue-200">Epidemiological Surveillance & Early Warning Dashboard (2569)</p>
                </div>
            </div>

            <button onclick="toggleMobileSidebar()" class="lg:hidden p-2 rounded-lg text-slate-300 hover:text-white hover:bg-slate-800">
                <i data-lucide="menu" class="w-6 h-6"></i>
            </button>
        </div>
    </header>

    <!-- 3. MAIN DASHBOARD CONTAINER -->
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6 flex-grow w-full">
        <div class="grid grid-cols-1 lg:grid-cols-4 gap-6">

            <!-- SIDEBAR CONTROLS (Left Panel) -->
            <aside id="sidebarPanel" class="lg:col-span-1 space-y-6">
                <!-- User Role Switcher -->
                <div class="bg-white rounded-2xl p-5 border border-slate-200 shadow-sm">
                    <h3 class="text-sm font-bold text-slate-900 uppercase tracking-wider mb-3 flex items-center space-x-2">
                        <i data-lucide="shield-check" class="w-4 h-4 text-blue-600"></i>
                        <span>สิทธิ์การเข้าถึงข้อมูล</span>
                    </h3>
                    <div class="space-y-2">
                        <label onclick="switchRole('public')" id="rolePublicLabel" class="flex items-center justify-between p-3 rounded-xl border-2 cursor-pointer transition-all border-blue-600 bg-blue-50/50">
                            <div class="flex items-center space-x-2.5">
                                <i data-lucide="users" class="w-4 h-4 text-blue-600"></i>
                                <span class="text-sm font-bold text-slate-800">ประชาชนทั่วไป</span>
                            </div>
                            <input type="radio" name="userRole" value="public" checked class="text-blue-600">
                        </label>
                        <label onclick="switchRole('staff')" id="roleStaffLabel" class="flex items-center justify-between p-3 rounded-xl border border-slate-200 cursor-pointer hover:bg-slate-50 transition-all">
                            <div class="flex items-center space-x-2.5">
                                <i data-lucide="stethoscope" class="w-4 h-4 text-slate-500"></i>
                                <span class="text-sm font-semibold text-slate-700">เจ้าหน้าที่ / บุคลากร</span>
                            </div>
                            <input type="radio" name="userRole" value="staff" class="text-blue-600">
                        </label>
                    </div>
                </div>

                <!-- Global Filters -->
                <div class="bg-white rounded-2xl p-5 border border-slate-200 shadow-sm space-y-4">
                    <h3 class="text-sm font-bold text-slate-900 uppercase tracking-wider flex items-center space-x-2 border-b border-slate-100 pb-2">
                        <i data-lucide="filter" class="w-4 h-4 text-blue-600"></i>
                        <span>ตัวกรองข้อมูลเฝ้าระวัง</span>
                    </h3>

                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">เลือกโรคระบาดตามฤดูกาล:</label>
                        <select id="diseaseSelect" onchange="onFilterChange()" class="w-full bg-slate-50 border border-slate-300 text-slate-800 text-sm rounded-xl p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none font-medium">
                            <option value="โรคไข้เลือดออก">🦟 โรคไข้เลือดออก (Dengue)</option>
                            <option value="โรคมือเท้าปาก">👶 โรคมือเท้าปาก (HFMD)</option>
                            <option value="โรคไข้หวัดใหญ่">🤒 โรคไข้หวัดใหญ่ (Flu)</option>
                            <option value="โรคปอดบวม/ปอดอักเสบ">🫁 โรคปอดบวม/ปอดอักเสบ</option>
                            <option value="โรคลมแดด (Heatstroke)">☀️ โรคลมแดด (Heatstroke)</option>
                            <option value="โรคอุจจาระร่วงเฉียบพลัน">💧 โรคอุจจาระร่วงเฉียบพลัน</option>
                        </select>
                    </div>

                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">เลือกเดือนประจำการ:</label>
                        <select id="monthSelect" onchange="onFilterChange()" class="w-full bg-slate-50 border border-slate-300 text-slate-800 text-sm rounded-xl p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none font-medium">
                            <option value="มกราคม">มกราคม</option>
                            <option value="กุมภาพันธ์">กุมภาพันธ์</option>
                            <option value="มีนาคม">มีนาคม</option>
                            <option value="เมษายน">เมษายน</option>
                            <option value="พฤษภาคม">พฤษภาคม</option>
                            <option value="มิถุนายน">มิถุนายน</option>
                            <option value="กรกฎาคม" selected>กรกฎาคม</option>
                            <option value="สิงหาคม">สิงหาคม</option>
                            <option value="กันยายน">กันยายน</option>
                            <option value="ตุลาคม">ตุลาคม</option>
                            <option value="พฤศจิกายน">พฤศจิกายน</option>
                            <option value="ธันวาคม">ธันวาคม</option>
                        </select>
                    </div>

                    <div class="p-3 bg-slate-50 rounded-xl border border-slate-200 text-xs text-slate-500 space-y-1">
                        <p class="font-semibold text-slate-700">💡 โมเดลเพดานเฝ้าระวัง:</p>
                        <p>ใช้อัลกอริทึม <span class="font-bold text-blue-600">Mean + 2SD</span> อ้างอิงสถิติย้อนหลัง 3 ปี (2566 - 2568) เพื่อประเมินสภาวะ Outbreak ปี 2569</p>
                    </div>
                </div>

                <!-- Quick Help Hotline Box -->
                <div class="bg-gradient-to-br from-slate-900 to-blue-950 text-white rounded-2xl p-5 shadow-md">
                    <h4 class="font-bold text-sm flex items-center space-x-2 text-blue-300 mb-2">
                        <i data-lucide="phone-call" class="w-4 h-4"></i>
                        <span>ช่องทางติดต่อฉุกเฉิน</span>
                    </h4>
                    <ul class="text-xs space-y-2 text-slate-300">
                        <li class="flex justify-between border-b border-slate-800 pb-1">
                            <span>สายด่วนกรมควบคุมโรค</span>
                            <strong class="text-white">1422</strong>
                        </li>
                        <li class="flex justify-between border-b border-slate-800 pb-1">
                            <span>เจ็บป่วยฉุกเฉิน (สพฉ.)</span>
                            <strong class="text-red-400">1669</strong>
                        </li>
                    </ul>
                </div>
            </aside>

            <!-- MAIN DISPLAY AREA (Right Panel) -->
            <main class="lg:col-span-3 space-y-6">

                <!-- Alert Dynamic Status Banner -->
                <div id="dynamicAlertBox" class="rounded-2xl p-4 md:p-5 border shadow-sm transition-all duration-300">
                    <!-- Dynamic Alert Content Injected via JS -->
                </div>

                <!-- VIEW 1: PUBLIC ACCESS ZONE -->
                <div id="publicZone" class="space-y-6">

                    <!-- Selected Disease Clinical Deep Dive -->
                    <div class="bg-white rounded-2xl p-6 border border-slate-200 shadow-sm space-y-6">
                        <div class="flex flex-col md:flex-row md:items-center justify-between border-b border-slate-100 pb-4 gap-2">
                            <div>
                                <span id="pubIcd10" class="text-xs font-bold text-blue-600 bg-blue-50 border border-blue-200 px-2.5 py-1 rounded-md">ICD-10</span>
                                <h2 id="pubDiseaseTitle" class="text-2xl font-bold text-slate-900 mt-1">ชื่อโรคระบาด</h2>
                            </div>
                            <span class="text-xs text-slate-400 flex items-center gap-1">
                                <i data-lucide="check-circle-2" class="w-4 h-4 text-emerald-500"></i>
                                ข้อมูลผ่านการรับรองโดยกรมควบคุมโรค
                            </span>
                        </div>

                        <!-- 2 Columns Clinical Info -->
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                            <div class="md:col-span-2 space-y-4">
                                <div class="bg-slate-50 p-4 rounded-xl border border-slate-200">
                                    <h4 class="text-sm font-bold text-blue-800 flex items-center space-x-2 mb-1.5">
                                        <i data-lucide="bug" class="w-4 h-4 text-blue-600"></i>
                                        <span>สาเหตุและพาหะแพร่กระจายเชื้อ</span>
                                    </h4>
                                    <p id="pubVector" class="text-sm text-slate-700 leading-relaxed"></p>
                                </div>

                                <div class="bg-slate-50 p-4 rounded-xl border border-slate-200">
                                    <h4 class="text-sm font-bold text-blue-800 flex items-center space-x-2 mb-1.5">
                                        <i data-lucide="thermometer-snowflake" class="w-4 h-4 text-blue-600"></i>
                                        <span>อาการสัญลักษณ์และข้อสังเกต</span>
                                    </h4>
                                    <p id="pubSymptoms" class="text-sm text-slate-700 leading-relaxed"></p>
                                </div>

                                <div class="bg-slate-50 p-4 rounded-xl border border-slate-200">
                                    <h4 class="text-sm font-bold text-blue-800 flex items-center space-x-2 mb-1.5">
                                        <i data-lucide="pill" class="w-4 h-4 text-blue-600"></i>
                                        <span>การวินิจฉัยและการรักษาเบื้องต้น</span>
                                    </h4>
                                    <p id="pubTreatment" class="text-sm text-slate-700 leading-relaxed"></p>
                                </div>

                                <div class="bg-slate-50 p-4 rounded-xl border border-slate-200">
                                    <h4 class="text-sm font-bold text-blue-800 flex items-center space-x-2 mb-1.5">
                                        <i data-lucide="map-pin" class="w-4 h-4 text-blue-600"></i>
                                        <span>ลักษณะทางระบาดวิทยาและพื้นที่เสี่ยง</span>
                                    </h4>
                                    <p id="pubEpidemiology" class="text-sm text-slate-700 leading-relaxed"></p>
                                </div>
                            </div>

                            <!-- Image & Important Guidelines Right Side -->
                            <div class="space-y-4">
                                <div class="bg-slate-100 rounded-xl overflow-hidden border border-slate-200 shadow-sm">
                                    <img id="pubSymptomImg" src="" alt="ภาพประกอบโรค" class="w-full h-48 object-cover">
                                    <div class="p-3 bg-white text-xs text-center text-slate-500 font-medium">
                                        ภาพประกอบทางการแพทย์เพื่อการสังเกตอาการ
                                    </div>
                                </div>

                                <div class="bg-gradient-to-br from-blue-600 to-sky-700 text-white p-5 rounded-xl shadow-md space-y-2">
                                    <h4 class="font-bold text-sm flex items-center space-x-2">
                                        <i data-lucide="syringe" class="w-4 h-4"></i>
                                        <span>คำแนะนำเรื่องวัคซีน</span>
                                    </h4>
                                    <p id="pubVaccine" class="text-xs text-blue-50 leading-relaxed"></p>
                                </div>
                            </div>
                        </div>
                    </div>

                </div>

                <!-- VIEW 2: MEDICAL STAFF ZONE (RESTRICTED) -->
                <div id="staffZone" class="hidden space-y-6">

                    <!-- Staff Passcode Auth Modal / Box -->
                    <div id="staffAuthBox" class="bg-white rounded-2xl p-8 border border-slate-200 shadow-sm text-center max-w-md mx-auto space-y-4">
                        <div class="w-14 h-14 bg-blue-100 text-blue-600 rounded-full flex items-center justify-center mx-auto">
                            <i data-lucide="lock" class="w-7 h-7"></i>
                        </div>
                        <div>
                            <h3 class="text-xl font-bold text-slate-900">ยืนยันตัวตนสำหรับบุคลากร</h3>
                            <p class="text-xs text-slate-500 mt-1">เข้าสู่ศูนย์สารสนเทศและเฝ้าระวังระบาดวิทยาเชิงลึก</p>
                        </div>
                        <div class="space-y-3 pt-2">
                            <input type="password" id="passcodeInput" placeholder="กรอกรหัสผ่าน (ทดสอบใช้: 1234)" class="w-full text-center bg-slate-50 border border-slate-300 text-slate-900 font-bold tracking-widest text-lg rounded-xl p-3 focus:ring-2 focus:ring-blue-500 focus:outline-none">
                            <button onclick="checkPasscode()" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 rounded-xl shadow-md transition-all">
                                เข้าสู่ระบบสารสนเทศ
                            </button>
                            <p id="passcodeError" class="text-xs text-red-600 hidden font-bold">❌ รหัสผ่านไม่ถูกต้อง กรุณาลองใหม่อีกครั้ง</p>
                        </div>
                    </div>

                    <!-- Staff Content (Shown after login) -->
                    <div id="staffContent" class="hidden space-y-6">

                        <!-- Staff Top Status & Logout -->
                        <div class="bg-emerald-50 border border-emerald-200 rounded-2xl p-4 flex flex-col sm:flex-row items-center justify-between gap-3">
                            <div class="flex items-center space-x-3 text-emerald-800">
                                <i data-lucide="check-circle" class="w-6 h-6 text-emerald-600 flex-shrink-0"></i>
                                <span class="text-xs md:text-sm font-medium">ยืนยันตัวตนสำเร็จ | สิทธิ์การเข้าใช้งาน: <strong>เจ้าหน้าที่ฝ่ายระบาดวิทยา / เวชกรรมสังคม</strong></span>
                            </div>
                            <button onclick="logoutStaff()" class="text-xs font-bold text-slate-600 hover:text-red-600 bg-white border border-slate-200 hover:border-red-200 px-3 py-1.5 rounded-lg shadow-sm transition-all">
                                ออกจากระบบ
                            </button>
                        </div>

                        <!-- KPI Surveillance Summary Metrics -->
                        <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                            <div class="bg-white p-4 rounded-xl border border-slate-200 border-l-4 border-l-blue-600 shadow-sm">
                                <span class="text-xs font-semibold text-slate-500 block">ผู้ป่วยล่าสุด (2569)</span>
                                <span id="kpiCurrent" class="text-2xl font-extrabold text-slate-900 mt-1 block">0</span>
                            </div>
                            <div class="bg-white p-4 rounded-xl border border-slate-200 border-l-4 border-l-amber-500 shadow-sm">
                                <span class="text-xs font-semibold text-slate-500 block">เพดานวิกฤต (Mean+2SD)</span>
                                <span id="kpiThreshold" class="text-2xl font-extrabold text-amber-600 mt-1 block">0</span>
                            </div>
                            <div class="bg-white p-4 rounded-xl border border-slate-200 border-l-4 border-l-slate-400 shadow-sm">
                                <span class="text-xs font-semibold text-slate-500 block">ค่าเฉลี่ยย้อนหลัง (66-68)</span>
                                <span id="kpiMean" class="text-2xl font-extrabold text-slate-700 mt-1 block">0</span>
                            </div>
                            <div class="bg-white p-4 rounded-xl border border-slate-200 border-l-4 border-l-emerald-600 shadow-sm">
                                <span class="text-xs font-semibold text-slate-500 block">เทียบสถิติปี 2568</span>
                                <span id="kpiDiff" class="text-2xl font-extrabold text-emerald-600 mt-1 block">0</span>
                            </div>
                        </div>

                        <!-- Epidemic Curve Interactive Chart -->
                        <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
                            <h3 class="text-lg font-bold text-slate-900 flex items-center space-x-2 border-b border-slate-100 pb-3">
                                <i data-lucide="line-chart" class="w-5 h-5 text-blue-600"></i>
                                <span>กราฟแสดงแนวโน้มสถิติรายเดือน (Epidemic Curve)</span>
                            </h3>
                            <div class="relative h-72 w-full">
                                <canvas id="epidemicChart"></canvas>
                            </div>
                        </div>

                    </div>

                </div>

            </main>

        </div>
    </div>

    <!-- FOOTER -->
    <footer class="bg-slate-900 text-slate-400 text-xs py-6 border-t border-slate-800 mt-12">
        <div class="max-w-7xl mx-auto px-4 text-center space-y-2">
            <p>© 2026 ระบบสารสนเทศและเฝ้าระวังโรคระบาดตามฤดูกาล. All rights reserved.</p>
            <p>แหล่งข้อมูลอ้างอิง: กรมควบคุมโรค กระทรวงสาธารณสุข (DDC) | องค์การอนามัยโลก (WHO)</p>
        </div>
    </footer>

    <!-- 4. JAVASCRIPT APPLICATION LOGIC -->
    <script>
        const diseasesList = [
            "โรคไข้เลือดออก", "โรคมือเท้าปาก", "โรคไข้หวัดใหญ่",
            "โรคปอดบวม/ปอดอักเสบ", "โรคลมแดด (Heatstroke)", "โรคอุจจาระร่วงเฉียบพลัน"
        ];
        const monthsList = [
            "มกราคม", "กุมภาพันธ์", "มีนาคม", "เมษายน", "พฤษภาคม", "มิถุนายน",
            "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม", "พฤศจิกายน", "ธันวาคม"
        ];

        const diseaseDatabase = {
            "โรคไข้เลือดออก": {
                icd10: "ICD-10: A90 (Dengue Fever)",
                vector: "เชื้อไวรัสเดงกี (Dengue Virus) โดยมียุงลายบ้านเป็นพาหะหลัก",
                symptoms: "ไข้สูงลอยเฉียบพลัน 2-7 วัน ปวดศีรษะ ปวดเบ้าตา ปวดเมื่อยตามตัว มีจุดเลือดออก",
                treatment: "เน้นการรักษาประคบประคอง ให้สารน้ำ/เกลือแร่ (ORS) เช็ดตัวลดไข้ ห้ามทานยาแอสไพริน",
                epidemiology: "พบการระบาดสูงช่วงฤดูฝน (พฤษภาคม - กันยายน)",
                vaccine: "💉 มีวัคซีนป้องกัน (Qdenga) แนะนำสำหรับผู้มีอายุ 4-60 ปี",
                img: "https://images.unsplash.com/photo-1584036561566-baf8f5f1b144?w=600&q=80"
            },
            "โรคมือเท้าปาก": {
                icd10: "ICD-10: B08.4 (HFMD)",
                vector: "เชื้อไวรัสกลุ่มเอนเทอโรไวรัส (Coxsackievirus A16, EV71)",
                symptoms: "มีไข้ เจ็บปาก มีแผลร้อนในตุ่มน้ำใสในปาก มีตุ่มแดงขึ้นตามฝ่ามือ ฝ่าเท้า",
                treatment: "รักษาตามอาการ ให้ยาลดไข้ ยาชาทาแผลในปาก ทานอาหารอ่อน/เย็น",
                epidemiology: "พบบ่อยในเด็กเล็กอายุต่ำกว่า 5 ปี มักระบาดช่วงเปิดเทอมฤดูฝน",
                vaccine: "💉 มีวัคซีนป้องกันไวรัส EV71 สำหรับเด็กอายุ 6 เดือน - 5 ปี",
                img: "https://images.unsplash.com/photo-1576091160399-112ba8d25d1d?w=600&q=80"
            },
            "โรคไข้หวัดใหญ่": {
                icd10: "ICD-10: J09-J11 (Influenza)",
                vector: "เชื้อไวรัสไข้หวัดใหญ่ สายพันธุ์ A และ B",
                symptoms: "ไข้สูงเฉียบพลัน ไอแห้ง ปวดศีรษะรุนแรง ปวดเมื่อยกล้ามเนื้อ",
                treatment: "พักผ่อน ดื่มน้ำอุ่นมากๆ กลุ่มเสี่ยงพิจารณาให้ยา Oseltamivir",
                epidemiology: "ระบาดสูงช่วงฤดูฝนและฤดูหนาว",
                vaccine: "💉 มีวัคซีนป้องกันไข้หวัดใหญ่ตามฤดูกาล แนะนำฉีดปีละ 1 ครั้ง",
                img: "https://images.unsplash.com/photo-1584515979956-d9f6e5d09982?w=600&q=80"
            },
            "โรคปอดบวม/ปอดอักเสบ": {
                icd10: "ICD-10: J12-J18 (Pneumonia)",
                vector: "เชื้อแบคทีเรีย Streptococcus pneumoniae หรือเชื้อไวรัส",
                symptoms: "ไอมีเสมหะข้น หายใจหอบเหนื่อย เจ็บหน้าอก ไข้สูง หนาวสั่น",
                treatment: "ติดเชื้อแบคทีเรียต้องได้รับยาปฏิชีวนะตามแพทย์สั่ง ให้ออกซิเจน",
                epidemiology: "พบอัตราป่วยสูงในผู้สูงอายุและเด็กเล็ก ช่วงอากาศเย็น",
                vaccine: "💉 มีวัคซีนป้องกันโรคติดเชื้อนิวโมคอคคัส (IPD)",
                img: "https://images.unsplash.com/photo-1530497610245-94d3c16cda28?w=600&q=80"
            },
            "โรคลมแดด (Heatstroke)": {
                icd10: "ICD-10: T67.0 (Heatstroke)",
                vector: "สภาวะอากาศร้อนจัด ทำให้อุณหภูมิกายสูงเกิน 40°C",
                symptoms: "อุณหภูมิกายสูงเกิน 40°C ผิวหนังแดง ร้อน แห้ง สับสน ชัก หมดสติ",
                treatment: "ภาวะฉุกเฉิน! ย้ายเข้าที่ร่ม ใช้ผ้าชุบน้ำเย็นประคบ รีบนำส่ง รพ.",
                epidemiology: "พบมากช่วงหน้าร้อน (มีนาคม - พฤษภาคม)",
                vaccine: "❌ ไม่มียาหรือวัคซีน เน้นดื่มน้ำมากๆ หลีกเลี่ยงแดดจัด",
                img: "https://images.unsplash.com/photo-1504384308090-c894fdcc538d?w=600&q=80"
            },
            "โรคอุจจาระร่วงเฉียบพลัน": {
                icd10: "ICD-10: A09 (Gastroenteritis)",
                vector: "เชื้อไวรัส (Rotavirus, Norovirus) หรือแบคทีเรีย ปนเปื้อนในอาหาร/น้ำ",
                symptoms: "ถ่ายเหลวเป็นน้ำหลายครั้ง ปวดท้องเกร็ง คลื่นไส้ อาเจียน",
                treatment: "จิบสารละลายน้ำเกลือแร่ (ORS) ห้ามกินยาหยุดถ่ายทันที",
                epidemiology: "พบระบาดได้ตลอดปี ไวรัสระบาดมากช่วงฤดูหนาว",
                vaccine: "💉 มีวัคซีนหยอดโรต้าสำหรับเด็กเล็ก",
                img: "https://images.unsplash.com/photo-1559839734-2b71ea197ec2?w=600&q=80"
            }
        };

        let masterDatabase = [];
        let epidemicChartInstance = null;

        function initDatabase() {
            masterDatabase = [];
            diseasesList.forEach(disease => {
                monthsList.forEach((month, index) => {
                    let monthIdx = index + 1;
                    let base = 20;
                    if (disease === "โรคไข้เลือดออก" && [5,6,7,8,9].includes(monthIdx)) base = 70;
                    else if (disease === "โรคไข้หวัดใหญ่" && [1,6,7,8,9,10].includes(monthIdx)) base = 90;

                    let seed = (disease.length * 10) + monthIdx;
                    let c2566 = Math.floor(base + (Math.sin(seed) * 10));
                    let c2567 = Math.floor(base * 1.1 + (Math.cos(seed) * 12));
                    let c2568 = Math.floor(base * 1.05 + (Math.sin(seed + 1) * 8));

                    let historical = [c2566, c2567, c2568];
                    let meanVal = historical.reduce((a, b) => a + b, 0) / 3;
                    let variance = historical.reduce((a, b) => a + Math.pow(b - meanVal, 2), 0) / 2;
                    let sdVal = Math.sqrt(variance);
                    let threshold = Math.ceil(meanVal + (2 * sdVal));
                    let c2569 = Math.floor(base * 1.25 + (Math.cos(seed + 2) * 15));

                    masterDatabase.push({
                        disease: disease, monthIdx: monthIdx, month: month,
                        c2566: c2566, c2567: c2567, c2568: c2568,
                        mean: parseFloat(meanVal.toFixed(1)), threshold: threshold, c2569: c2569
                    });
                });
            });
        }

        function closeEmergencyModal() { document.getElementById('emergencyModal').classList.add('hidden'); }
        function toggleMobileSidebar() { document.getElementById('sidebarPanel').classList.toggle('hidden'); }

        function switchRole(role) {
            if (role === 'public') {
                document.getElementById('publicZone').classList.remove('hidden');
                document.getElementById('staffZone').classList.add('hidden');
            } else {
                document.getElementById('publicZone').classList.add('hidden');
                document.getElementById('staffZone').classList.remove('hidden');
            }
        }

        function checkPasscode() {
            if (document.getElementById('passcodeInput').value === '1234') {
                document.getElementById('staffAuthBox').classList.add('hidden');
                document.getElementById('staffContent').classList.remove('hidden');
                renderStaffView();
            } else {
                document.getElementById('passcodeError').classList.remove('hidden');
            }
        }

        function logoutStaff() {
            document.getElementById('staffAuthBox').classList.remove('hidden');
            document.getElementById('staffContent').classList.add('hidden');
        }

        function onFilterChange() { renderDashboard(); }

        function renderDashboard() {
            const disease = document.getElementById('diseaseSelect').value;
            const month = document.getElementById('monthSelect').value;
            const record = masterDatabase.find(d => d.disease === disease && d.month === month);
            if (!record) return;

            renderPublicView(disease);
            renderStaffView();
            renderAlertBanner(record);
        }

        function renderAlertBanner(record) {
            const box = document.getElementById('dynamicAlertBox');
            if (record.c2569 >= record.threshold) {
                box.className = "rounded-2xl p-4 md:p-5 border shadow-sm transition-all duration-300 bg-red-50 border-red-200 text-red-900";
                box.innerHTML = `
                    <div class="flex items-start space-x-3">
                        <i data-lucide="alert-triangle" class="w-6 h-6 text-red-600 flex-shrink-0 mt-0.5"></i>
                        <div>
                            <h4 class="font-bold text-base text-red-800">🚨 OUTBREAK ALERT: สภาวะแพร่ระบาดระเบิดตัว (${record.disease})</h4>
                            <p class="text-xs md:text-sm mt-1">จำนวนผู้ป่วยเดือน ${record.month} สถิติล่าสุด <strong>${record.c2569} ราย</strong> ทะลุเพดานวิกฤต (Mean+2SD: <strong>${record.threshold} ราย</strong>)</p>
                        </div>
                    </div>`;
            } else {
                box.className = "rounded-2xl p-4 md:p-5 border shadow-sm transition-all duration-300 bg-emerald-50 border-emerald-200 text-emerald-900";
                box.innerHTML = `
                    <div class="flex items-start space-x-3">
                        <i data-lucide="check-circle-2" class="w-6 h-6 text-emerald-600 flex-shrink-0 mt-0.5"></i>
                        <div>
                            <h4 class="font-bold text-base text-emerald-800">✅ สถานการณ์ปกติ (${record.disease})</h4>
                            <p class="text-xs md:text-sm mt-1">จำนวนผู้ป่วยเดือน ${record.month} สถิติล่าสุด <strong>${record.c2569} ราย</strong> ยังอยู่ต่ำกว่าเกณฑ์เพดานเฝ้าระวัง (${record.threshold} ราย)</p>
                        </div>
                    </div>`;
            }
            lucide.createIcons();
        }

        function renderPublicView(diseaseName) {
            const info = diseaseDatabase[diseaseName];
            if (!info) return;
            document.getElementById('pubIcd10').innerText = info.icd10;
            document.getElementById('pubDiseaseTitle').innerText = diseaseName;
            document.getElementById('pubVector').innerText = info.vector;
            document.getElementById('pubSymptoms').innerText = info.symptoms;
            document.getElementById('pubTreatment').innerText = info.treatment;
            document.getElementById('pubEpidemiology').innerText = info.epidemiology;
            document.getElementById('pubVaccine').innerText = info.vaccine;
            document.getElementById('pubSymptomImg').src = info.img;
        }

        function renderStaffView() {
            const disease = document.getElementById('diseaseSelect').value;
            const month = document.getElementById('monthSelect').value;
            const record = masterDatabase.find(d => d.disease === disease && d.month === month);
            if (!record) return;

            document.getElementById('kpiCurrent').innerText = record.c2569 + " ราย";
            document.getElementById('kpiThreshold').innerText = record.threshold + " ราย";
            document.getElementById('kpiMean').innerText = record.mean + " ราย";
            document.getElementById('kpiDiff').innerText = (record.c2569 - record.c2568) + " ราย";

            const diseaseYearData = masterDatabase.filter(d => d.disease === disease).sort((a,b) => a.monthIdx - b.monthIdx);

            const ctx = document.getElementById('epidemicChart').getContext('2d');
            if (epidemicChartInstance) epidemicChartInstance.destroy();

            epidemicChartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: monthsList,
                    datasets: [
                        {
                            label: 'ปี 2568 (ปีก่อน)',
                            data: diseaseYearData.map(d => d.c2568),
                            borderColor: '#94A3B8',
                            borderDash: [5, 5],
                            tension: 0.3
                        },
                        {
                            label: 'ปี 2569 (ปัจจุบัน)',
                            data: diseaseYearData.map(d => d.c2569),
                            borderColor: '#0284C7',
                            backgroundColor: 'rgba(2, 132, 199, 0.1)',
                            borderWidth: 3,
                            fill: true,
                            tension: 0.3
                        },
                        {
                            label: 'เพดานวิกฤต (Mean+2SD)',
                            data: diseaseYearData.map(d => d.threshold),
                            borderColor: '#EF4444',
                            borderWidth: 2,
                            borderDash: [3, 3],
                            pointRadius: 0
                        }
                    ]
                },
                options: { responsive: true, maintainAspectRatio: false }
            });
        }

        window.onload = function() {
            initDatabase();
            renderDashboard();
            lucide.createIcons();
        };
    </script>
</body>
</html>

import subprocess
import time

# 1. รัน Python HTTP Server บนพอร์ต 8000 ไว้เบื้องหลัง
server_process = subprocess.Popen(["python3", "-m", "http.server", "8000"])

# 2. หน่วงเวลา 2 วินาทีให้ Server เปิดพร้อมใช้งาน
time.sleep(2)

# 3. รัน Cloudflare Tunnel เพื่อสร้างลิงก์เข้าสู่เว็บ
!cloudflared tunnel --url http://localhost:8000
