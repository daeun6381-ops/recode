# recode
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>1등급 제작소</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://cdn.jsdelivr.net/gh/orioncactus/pretendard/dist/web/static/pretendard.css');
        
        body {
            font-family: 'Pretendard', 'Malgun Gothic', '맑은 고딕', sans-serif;
            background-color: #d1d5db; 
        }

        .paper-container {
            background-color: white;
            margin: 2rem auto;
            padding: 3rem 2rem;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            min-height: 297mm; 
            max-width: 5xl;
        }

        .neis-table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 2.5rem;
            border-top: 2px solid #000;
            border-bottom: 2px solid #000;
            font-size: 0.95rem;
        }
        .neis-table th, .neis-table td {
            border: 1px solid #9ca3af; 
            padding: 0.5rem;
            vertical-align: middle;
        }
        .neis-table th {
            background-color: #f3f4f6; 
            font-weight: bold;
            text-align: center;
            color: #111827;
        }
        
        .section-title {
            font-size: 1.1rem;
            font-weight: bold;
            color: #000;
            margin-bottom: 0.5rem;
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
        }

        .smart-textarea-container {
            position: relative;
            width: 100%;
            height: 180px;
            background-color: white;
            overflow: hidden;
        }

        .smart-textarea-container:focus-within {
            outline: 2px solid rgba(59, 130, 246, 0.5);
            outline-offset: -2px;
        }

        .highlight-backdrop, .highlight-textarea {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            padding: 0.75rem;
            font-family: inherit;
            font-size: 0.95rem;
            line-height: 1.6;
            border: none;
            outline: none;
            white-space: pre-wrap;
            word-wrap: break-word;
            overflow-y: auto;
            resize: none;
            margin: 0;
        }

        .highlight-backdrop {
            z-index: 1;
            color: transparent;
            pointer-events: none;
        }

        .highlight-textarea {
            z-index: 2;
            background-color: transparent;
            color: #111827;
        }

        .highlight-backdrop mark {
            background-color: #bae6fd;
            color: transparent;
            border-radius: 2px;
        }

        .highlight-textarea::-webkit-scrollbar, .highlight-backdrop::-webkit-scrollbar {
            width: 8px;
        }
        .highlight-textarea::-webkit-scrollbar-thumb {
            background-color: #d1d5db;
            border-radius: 4px;
        }

        /* 틀림 애니메이션 (흔들기) */
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            20%, 60% { transform: translateX(-5px); }
            40%, 80% { transform: translateX(5px); }
        }
        .animate-shake {
            animation: shake 0.4s ease-in-out;
        }

        /* 패턴 락 컨테이너 스크롤 방지 */
        .pattern-container {
            touch-action: none; 
            user-select: none;
        }

        @media print {
            body { background-color: white; }
            .paper-container { margin: 0; padding: 0; box-shadow: none; max-width: 100%; }
            .no-print { display: none !important; }
            .neis-table { page-break-inside: avoid; }
        }
    </style>
</head>
<body class="text-gray-900 antialiased min-h-screen relative">

    <!-- 로딩 화면 -->
    <div id="loading-screen" class="min-h-screen flex flex-col items-center justify-center p-4 bg-gray-100 fixed inset-0 z-[200]">
        <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-blue-800 mb-4"></div>
        <h2 class="text-xl font-bold text-gray-800">서버 통신 중...</h2>
        <p class="text-gray-500 text-sm mt-2">안전하게 데이터를 불러오고 있습니다.</p>
    </div>

    <!-- 로그인/회원가입 화면 -->
    <div id="auth-screen" class="hidden min-h-screen flex items-center justify-center p-4 bg-gray-100 relative z-[150]">
        <div class="bg-white p-10 rounded-lg shadow-xl max-w-sm w-full text-center border-t-4 border-gray-800">
            <h1 class="text-2xl font-extrabold mb-1">1등급 제작소</h1>
            <p class="text-gray-500 mb-8 text-sm">어디서든 내 계정으로 로그인하세요</p>
            
            <input type="email" id="email-input" placeholder="이메일 (아이디)" class="w-full mb-3 p-3 border border-gray-300 rounded focus:outline-none focus:border-blue-500 text-sm">
            <input type="password" id="pwd-input" placeholder="비밀번호 (6자 이상)" class="w-full mb-6 p-3 border border-gray-300 rounded focus:outline-none focus:border-blue-500 text-sm">
            
            <div class="flex gap-2 mb-6">
                <button onclick="handleLogin()" class="w-1/2 py-3 bg-gray-800 hover:bg-gray-700 text-white font-bold rounded transition shadow-sm">로그인</button>
                <button onclick="handleSignup()" class="w-1/2 py-3 bg-blue-600 hover:bg-blue-500 text-white font-bold rounded transition shadow-sm">가입하기</button>
            </div>

            <div class="border-t border-gray-200 mt-2 pt-4">
                <button onclick="handleGuestLogin()" class="text-xs text-gray-400 hover:text-gray-600 font-medium">로그인 없이 기기에 임시저장 (게스트)</button>
            </div>
        </div>
    </div>

    <!-- 🔒 잠금 화면 (보안 설정 시 등장) -->
    <div id="lock-screen" class="hidden min-h-screen flex items-center justify-center p-4 bg-gray-900 text-white fixed inset-0 z-[180]">
        <div class="text-center w-full max-w-sm">
            <div class="mb-6 flex justify-center">
                <svg class="w-16 h-16 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"></path></svg>
            </div>
            <h2 class="text-2xl font-bold mb-2">보안 잠금</h2>
            <p id="lock-screen-msg" class="text-gray-400 text-sm mb-8">내 생기부를 보호하기 위해 잠금을 해제하세요.</p>

            <!-- PIN 해제 UI -->
            <div id="lock-pin-ui" class="hidden">
                <div class="flex justify-center gap-4 mb-8" id="pin-dots">
                    <div class="w-4 h-4 rounded-full border-2 border-gray-400"></div>
                    <div class="w-4 h-4 rounded-full border-2 border-gray-400"></div>
                    <div class="w-4 h-4 rounded-full border-2 border-gray-400"></div>
                    <div class="w-4 h-4 rounded-full border-2 border-gray-400"></div>
                </div>
                <div class="grid grid-cols-3 gap-4 max-w-[250px] mx-auto text-xl font-bold">
                    <button onclick="pressPin('1')" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition">1</button>
                    <button onclick="pressPin('2')" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition">2</button>
                    <button onclick="pressPin('3')" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition">3</button>
                    <button onclick="pressPin('4')" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition">4</button>
                    <button onclick="pressPin('5')" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition">5</button>
                    <button onclick="pressPin('6')" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition">6</button>
                    <button onclick="pressPin('7')" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition">7</button>
                    <button onclick="pressPin('8')" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition">8</button>
                    <button onclick="pressPin('9')" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition">9</button>
                    <div class="py-4"></div>
                    <button onclick="pressPin('0')" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition">0</button>
                    <button onclick="clearPin()" class="py-4 bg-gray-800 rounded-full hover:bg-gray-700 transition flex justify-center items-center">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2M3 12l6.414 6.414a2 2 0 001.414.586H19a2 2 0 002-2V7a2 2 0 00-2-2h-8.172a2 2 0 00-1.414.586L3 12z"></path></svg>
                    </button>
                </div>
            </div>

            <!-- Pattern 해제 UI -->
            <div id="lock-pattern-ui" class="hidden">
                <div id="unlock-pattern-container" class="pattern-container grid grid-cols-3 gap-6 p-6 bg-gray-800 rounded-2xl inline-block mx-auto mb-6">
                    <!-- JS로 동적 생성됨 -->
                </div>
            </div>

            <div class="mt-10">
                <button onclick="handleLogout()" class="text-sm text-gray-500 hover:text-white underline">계정 로그아웃</button>
            </div>
        </div>
    </div>

    <!-- 학년 선택 화면 -->
    <div id="intro-screen" class="hidden min-h-screen flex items-center justify-center p-4 bg-gray-100 relative">
        <div class="absolute top-4 right-4 text-right flex items-start gap-2">
            <!-- 개인 서버 연동 시 나타나는 유저 정보 -->
            <div id="user-info-display" class="hidden text-right">
                <div class="text-xs font-bold text-gray-700 mb-1">👤 <span id="user-email-display"></span></div>
                <button onclick="handleLogout()" class="text-xs bg-gray-200 hover:bg-gray-300 text-gray-800 px-3 py-1 rounded font-bold transition">로그아웃</button>
            </div>
            <!-- 채팅창 환경 뱃지 -->
            <div id="auto-cloud-badge" class="bg-blue-100 text-blue-800 text-xs font-bold px-3 py-1.5 rounded-full shadow-sm mt-1">
                ☁️ 자동 연동됨
            </div>
            <!-- 보안 설정 버튼 -->
            <button onclick="openSecuritySettings()" class="p-2 bg-white rounded-full shadow hover:bg-gray-50 transition" title="보안 설정">
                <svg class="w-5 h-5 text-gray-700" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
            </button>
        </div>
        
        <div class="bg-white p-10 rounded-lg shadow-xl max-w-lg w-full text-center border-t-4 border-gray-800 mt-8">
            <h1 class="text-3xl font-extrabold mb-2 tracking-tight">학교생활기록부</h1>
            <p class="text-gray-600 mb-8 font-medium">관리용 생기부 에디터 (자동 클라우드 저장)</p>
            
            <h2 class="text-lg font-bold mb-6 text-gray-800">작성할 학년을 선택하십시오.</h2>
            <div class="grid grid-cols-1 gap-3 sm:grid-cols-3">
                <button onclick="startApp(1)" class="py-3 px-6 bg-white border-2 border-gray-300 hover:border-gray-800 hover:bg-gray-50 font-bold rounded transition">1학년</button>
                <button onclick="startApp(2)" class="py-3 px-6 bg-white border-2 border-gray-300 hover:border-gray-800 hover:bg-gray-50 font-bold rounded transition">2학년</button>
                <button onclick="startApp(3)" class="py-3 px-6 bg-white border-2 border-gray-300 hover:border-gray-800 hover:bg-gray-50 font-bold rounded transition">3학년</button>
            </div>
        </div>
    </div>

    <!-- 메인 에디터 화면 -->
    <div id="editor-screen" class="hidden">
        
        <!-- 상단 고정 컨트롤 바 -->
        <div class="sticky top-0 z-50 bg-gray-800 text-white p-3 flex justify-between items-center no-print shadow-md">
            <div class="flex items-center">
                <span class="font-bold mr-4 whitespace-nowrap">1등급 제작소</span>
                <span id="grade-badge" class="bg-gray-600 text-xs px-2 py-1 rounded whitespace-nowrap"></span>
                <span class="ml-4 text-xs text-blue-300 hidden md:inline font-bold">☁️ 변경사항 실시간 저장 중...</span>
            </div>
            <div class="flex items-center space-x-2">
                <button onclick="openSecuritySettings()" class="px-2 py-1.5 bg-gray-700 hover:bg-gray-600 rounded transition" title="보안 설정">
                    <svg class="w-4 h-4 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
                </button>
                <button onclick="resetApp()" class="px-3 py-1.5 bg-gray-700 hover:bg-gray-600 text-xs font-bold rounded transition whitespace-nowrap">학년 다시선택</button>
                <button id="export-btn" onclick="exportToDocument()" class="px-3 py-1.5 bg-green-600 hover:bg-green-500 text-xs font-bold rounded transition inline-flex items-center gap-1 whitespace-nowrap">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
                    <span class="hidden sm:inline">문서 다운로드 (한글/워드)</span>
                    <span class="sm:hidden">문서 다운로드</span>
                </button>
                <button onclick="saveData()" class="px-3 py-1.5 bg-blue-600 hover:bg-blue-500 text-xs font-bold rounded transition whitespace-nowrap">클라우드 저장</button>
                <button id="logout-btn" onclick="handleLogout()" class="hidden px-3 py-1.5 bg-red-600 hover:bg-red-500 text-xs font-bold rounded transition whitespace-nowrap">로그아웃</button>
            </div>
        </div>

        <div class="paper-container relative mx-auto max-w-5xl" id="document-content">
            <h1 class="text-3xl font-extrabold text-center mb-10 tracking-widest" style="font-family: 'Batang', '바탕', serif;">학교생활기록부</h1>

            <!-- 1. 수상경력 -->
            <section>
                <div class="section-title">
                    <span>▣ 4. 수상경력</span>
                    <button onclick="addAward()" class="text-xs font-normal bg-white border border-gray-400 px-2 py-1 hover:bg-gray-50 no-print">+ 수상 추가</button>
                </div>
                <table class="neis-table">
                    <colgroup>
                        <col style="width: 35%">
                        <col style="width: 30%">
                        <col style="width: 20%">
                        <col style="width: 15%" class="no-print">
                    </colgroup>
                    <thead>
                        <tr>
                            <th>수상명</th>
                            <th>수여기관</th>
                            <th>연월일</th>
                            <th class="no-print">관리</th>
                        </tr>
                    </thead>
                    <tbody id="awards-list"></tbody>
                </table>
            </section>

            <!-- 2. 봉사활동실적 -->
            <section>
                <div class="section-title">
                    <span>▣ 봉사활동실적</span>
                    <button onclick="addVolunteer()" class="text-xs font-normal bg-white border border-gray-400 px-2 py-1 hover:bg-gray-50 no-print">+ 봉사 추가</button>
                </div>
                <table class="neis-table">
                    <colgroup>
                        <col style="width: 15%">
                        <col style="width: 30%">
                        <col style="width: 35%">
                        <col style="width: 10%">
                        <col style="width: 10%" class="no-print">
                    </colgroup>
                    <thead>
                        <tr>
                            <th>연월일</th>
                            <th>장소 또는 주관기관명</th>
                            <th>활동내용</th>
                            <th>시간</th>
                            <th class="no-print">관리</th>
                        </tr>
                    </thead>
                    <tbody id="volunteer-list"></tbody>
                </table>
            </section>

            <!-- 3. 창의적 체험활동상황 -->
            <section>
                <div class="section-title">
                    <span>▣ 7. 창의적 체험활동상황</span>
                </div>
                <table class="neis-table">
                    <colgroup>
                        <col style="width: 15%">
                        <col style="width: 85%">
                    </colgroup>
                    <thead>
                        <tr>
                            <th>영역</th>
                            <th>특기사항</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td class="font-bold text-center bg-gray-50">자율활동</td>
                            <td class="p-0 relative align-top">
                                <div class="flex justify-end p-1 bg-gray-50 border-b border-gray-200 no-print">
                                    <span class="text-xs font-medium text-gray-500" id="counter-act-1">0 / 500자</span>
                                </div>
                                <div id="container-act-1" class="smart-textarea-container" data-limit="500"></div>
                            </td>
                        </tr>
                        <tr>
                            <td class="font-bold text-center bg-gray-50">동아리활동</td>
                            <td class="p-0 relative align-top">
                                <div class="flex justify-end p-1 bg-gray-50 border-b border-gray-200 no-print">
                                    <span class="text-xs font-medium text-gray-500" id="counter-act-2">0 / 500자</span>
                                </div>
                                <div id="container-act-2" class="smart-textarea-container" data-limit="500"></div>
                            </td>
                        </tr>
                        <tr>
                            <td class="font-bold text-center bg-gray-50">진로활동</td>
                            <td class="p-0 relative align-top">
                                <div class="flex justify-end p-1 bg-gray-50 border-b border-gray-200 no-print">
                                    <span class="text-xs font-medium text-gray-500" id="counter-act-3">0 / 700자</span>
                                </div>
                                <div id="container-act-3" class="smart-textarea-container" data-limit="700"></div>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </section>

            <!-- 4. 교과학습발달상황 -->
            <section>
                <div class="section-title">
                    <span>▣ 8. 교과학습발달상황 <span class="text-sm font-normal text-gray-600 ml-2">(세부능력 및 특기사항)</span></span>
                    <button onclick="addSubject()" class="text-xs font-normal bg-white border border-gray-400 px-2 py-1 hover:bg-gray-50 no-print">+ 과목 추가</button>
                </div>
                <table class="neis-table">
                    <colgroup>
                        <col style="width: 15%">
                        <col style="width: 77%">
                        <col style="width: 8%" class="no-print">
                    </colgroup>
                    <thead>
                        <tr>
                            <th>과목</th>
                            <th>세부능력 및 특기사항</th>
                            <th class="no-print">관리</th>
                        </tr>
                    </thead>
                    <tbody id="subjects-list"></tbody>
                </table>
            </section>

            <!-- 5. 행동특성 및 종합의견 -->
            <section>
                <div class="section-title">
                    <span>▣ 10. 행동특성 및 종합의견</span>
                </div>
                <table class="neis-table">
                    <colgroup>
                        <col style="width: 15%">
                        <col style="width: 85%">
                    </colgroup>
                    <tbody>
                        <tr>
                            <td class="font-bold text-center bg-gray-50">종합의견</td>
                            <td class="p-0 relative align-top">
                                <div class="flex justify-end p-1 bg-gray-50 border-b border-gray-200 no-print">
                                    <span class="text-xs font-medium text-gray-500" id="counter-behavior">0 / 500자</span>
                                </div>
                                <div id="container-behavior" class="smart-textarea-container" data-limit="500" style="height: 250px;"></div>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </section>

        </div>
    </div>

    <!-- ⚙️ 보안 설정 모달 -->
    <div id="security-modal" class="fixed inset-0 bg-black bg-opacity-50 z-[200] hidden items-center justify-center p-4">
        <div class="bg-white p-6 rounded-lg shadow-xl max-w-sm w-full">
            <h3 class="text-xl font-bold mb-4 flex items-center">
                <svg class="w-6 h-6 mr-2 text-gray-700" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"></path></svg>
                보안 설정 (앱 잠금)
            </h3>
            <p class="text-sm text-gray-500 mb-6">앱을 켤 때 생기부 내용을 보호할 수 있습니다.</p>
            
            <div class="mb-6">
                <label class="block text-sm font-bold text-gray-700 mb-2">잠금 방식</label>
                <select id="lock-type-select" onchange="toggleLockSetupUI()" class="w-full p-2 border border-gray-300 rounded focus:border-blue-500 focus:outline-none bg-white">
                    <option value="none">사용 안 함 (자동 진입)</option>
                    <option value="pin">4자리 비밀번호 (PIN)</option>
                    <option value="pattern">비밀 패턴</option>
                </select>
            </div>

            <!-- PIN 설정 UI -->
            <div id="setup-pin-ui" class="hidden mb-6 bg-gray-50 p-4 rounded-lg border border-gray-200">
                <label class="block text-sm font-bold text-gray-700 mb-2">새 비밀번호 입력 (4자리 숫자)</label>
                <input type="password" id="setup-pin-input" maxlength="4" placeholder="예: 1234" pattern="\d*" class="w-full p-3 border border-gray-300 rounded text-center text-xl tracking-[1em] focus:border-blue-500 focus:outline-none">
            </div>

            <!-- Pattern 설정 UI -->
            <div id="setup-pattern-ui" class="hidden mb-6 bg-gray-50 p-4 rounded-lg border border-gray-200 text-center">
                <label class="block text-sm font-bold text-gray-700 mb-2">새 패턴 그리기 (점 위를 드래그)</label>
                <div id="setup-pattern-container" class="pattern-container grid grid-cols-3 gap-4 p-4 bg-white border border-gray-200 rounded-xl inline-block mx-auto">
                    <!-- JS로 동적 생성 -->
                </div>
                <p id="setup-pattern-msg" class="text-xs mt-2 text-blue-600 font-bold h-4"></p>
                <button onclick="clearSetupPattern()" class="mt-2 text-xs text-gray-500 hover:text-gray-800 underline">패턴 다시 그리기</button>
            </div>

            <div class="flex justify-end gap-2">
                <button onclick="closeSecuritySettings()" class="px-4 py-2 bg-gray-200 hover:bg-gray-300 rounded font-bold text-sm transition text-gray-800">취소</button>
                <button onclick="saveSecuritySettings()" class="px-4 py-2 bg-blue-600 hover:bg-blue-500 text-white rounded font-bold text-sm transition">저장 적용</button>
            </div>
        </div>
    </div>

    <!-- 토스트 알림 / 에러 / 기존 모달 등 -->
    <div id="toast" class="fixed bottom-4 left-1/2 transform -translate-x-1/2 bg-gray-800 text-white px-4 py-2 rounded shadow-lg opacity-0 pointer-events-none transition-opacity duration-300 z-[400]">
        저장되었습니다.
    </div>

    <div id="confirm-modal" class="fixed inset-0 bg-black bg-opacity-50 z-[300] hidden items-center justify-center p-4">
        <div class="bg-white p-6 rounded-lg shadow-xl max-w-sm w-full text-center">
            <h3 class="text-lg font-bold mb-4">학년 변경</h3>
            <p class="text-gray-600 mb-6 text-sm">학년을 변경하시겠습니까?<br>현재 화면의 내용은 자동으로 클라우드에 저장됩니다.</p>
            <div class="flex justify-center space-x-3">
                <button onclick="closeConfirmModal()" class="px-4 py-2 bg-gray-200 hover:bg-gray-300 rounded font-bold text-sm transition text-gray-800">취소</button>
                <button onclick="executeResetApp()" class="px-4 py-2 bg-gray-800 hover:bg-gray-700 text-white rounded font-bold text-sm transition">확인</button>
            </div>
        </div>
    </div>

    <div id="error-modal" class="fixed inset-0 bg-black bg-opacity-50 z-[300] hidden items-center justify-center p-4">
        <div class="bg-white p-6 rounded-lg shadow-xl max-w-md w-full text-center">
            <div class="text-red-500 mb-4">
                <svg class="w-12 h-12 mx-auto" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"></path></svg>
            </div>
            <h3 class="text-lg font-bold mb-2">안내 사항</h3>
            <p id="error-modal-msg" class="text-gray-600 mb-6 text-sm whitespace-pre-wrap break-words leading-relaxed"></p>
            <button onclick="closeErrorModal()" class="px-6 py-2 bg-gray-800 hover:bg-gray-700 text-white rounded font-bold text-sm transition">확인</button>
        </div>
    </div>

    <!-- Firebase 및 비즈니스 로직 -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInWithEmailAndPassword, createUserWithEmailAndPassword, signInWithCustomToken, signInAnonymously, onAuthStateChanged, signOut, setPersistence, browserLocalPersistence } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // ============================================================================
        // 💡 [나만의 파이어베이스 설정 영역]
        // ============================================================================
        const MY_FIREBASE_CONFIG = {
            apiKey: "AIzaSyA30iujeXRiP1iJW1db_2WIttKJ3sO4SAI",
            authDomain: "space1-b521a.firebaseapp.com",
            projectId: "space1-b521a",
            storageBucket: "space1-b521a.firebasestorage.app",
            messagingSenderId: "235223237323",
            appId: "1:235223237323:web:a195263242c149e265514e",
            measurementId: "G-HJ34NT7FDV"
        };

        let currentGrade = null;
        let textareasMap = new Map();
        
        let app, auth, db, currentUser;
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        
        // --- 보안(앱 잠금) 상태 관리 변수 ---
        let securitySettings = { type: 'none', code: '' }; 
        let pendingTargetGrade = null; 
        
        // --- 패턴 락 그리기 상태 변수 ---
        let isDrawingPattern = false;
        let currentPatternCode = [];
        let activePatternContainerId = null;

        const defaultSubjects = {
            1: ['국어', '수학', '영어', '한국사', '통합사회', '통합과학', '과학탐구실험'],
            2: ['문학', '독서', '수학I', '수학II', '영어I', '영어II', '물리학I', '화학I', '생명과학I'],
            3: ['화법과 작문', '미적분', '확률과 통계', '심화국어', '영어독해와 작문', '사회·문화', '지구과학II']
        };

        // 1. Firebase 초기화
        async function initFirebase() {
            try {
                let configToUse = null;
                let isCanvasEnv = false;

                if (MY_FIREBASE_CONFIG) {
                    configToUse = MY_FIREBASE_CONFIG;
                } else if (typeof __firebase_config !== 'undefined') {
                    configToUse = JSON.parse(__firebase_config);
                    isCanvasEnv = true;
                }

                if (configToUse) {
                    app = initializeApp(configToUse);
                    auth = getAuth(app);
                    db = getFirestore(app);

                    if (!isCanvasEnv) {
                        try {
                            await setPersistence(auth, browserLocalPersistence);
                        } catch (e) {
                            console.error("자동 로그인 설정 오류:", e);
                        }
                    }

                    if (isCanvasEnv) {
                        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                            await signInWithCustomToken(auth, __initial_auth_token);
                        } else {
                            await signInAnonymously(auth);
                        }
                    }

                    // 인증 상태 처리
                    onAuthStateChanged(auth, async (user) => {
                        if (user) {
                            if (!isCanvasEnv && user.isAnonymous) {
                                await signOut(auth);
                                return; 
                            }

                            currentUser = user;
                            document.getElementById('auth-screen').classList.add('hidden');
                            
                            if (!isCanvasEnv) {
                                document.getElementById('auto-cloud-badge').classList.add('hidden');
                                document.getElementById('user-info-display').classList.remove('hidden');
                                document.getElementById('logout-btn').classList.remove('hidden');
                                document.getElementById('user-email-display').textContent = user.isAnonymous ? '게스트' : user.email;
                            }

                            if (document.getElementById('editor-screen').classList.contains('hidden')) {
                                let lastGrade = null;
                                
                                try {
                                    if (db) {
                                        const prefRef = doc(db, 'artifacts', appId, 'users', user.uid, 'settings', 'preferences');
                                        const secRef = doc(db, 'artifacts', appId, 'users', user.uid, 'settings', 'security');
                                        
                                        const [prefSnap, secSnap] = await Promise.all([getDoc(prefRef), getDoc(secRef)]);
                                        
                                        if (prefSnap.exists() && prefSnap.data().lastGrade) {
                                            lastGrade = prefSnap.data().lastGrade;
                                        }
                                        if (secSnap.exists() && secSnap.data().type) {
                                            securitySettings = secSnap.data();
                                        }
                                    }
                                } catch (e) {
                                    console.error("설정 로드 에러:", e);
                                }

                                if (!lastGrade) {
                                    const localPref = localStorage.getItem(`pocket-sgb-${user.uid}-last-grade`);
                                    if (localPref) lastGrade = parseInt(localPref);
                                }
                                
                                const localSec = localStorage.getItem(`pocket-sgb-${user.uid}-security`);
                                if (localSec) {
                                    const parsed = JSON.parse(localSec);
                                    if(parsed.type !== 'none') securitySettings = parsed;
                                }

                                document.getElementById('loading-screen').classList.add('hidden');

                                if (securitySettings.type !== 'none' && securitySettings.code) {
                                    pendingTargetGrade = lastGrade; 
                                    showLockScreen();
                                } else {
                                    if (lastGrade) {
                                        await startApp(lastGrade);
                                    } else {
                                        document.getElementById('intro-screen').classList.remove('hidden');
                                    }
                                }
                            } else {
                                document.getElementById('loading-screen').classList.add('hidden');
                            }
                        } else {
                            currentUser = null;
                            securitySettings = { type: 'none', code: '' };
                            document.getElementById('loading-screen').classList.add('hidden');
                            document.getElementById('intro-screen').classList.add('hidden');
                            document.getElementById('editor-screen').classList.add('hidden');
                            document.getElementById('lock-screen').classList.add('hidden');
                            document.getElementById('auth-screen').classList.remove('hidden');
                        }
                    });
                } else {
                    currentUser = { uid: 'local-tester' };
                    document.getElementById('loading-screen').classList.add('hidden');
                    document.getElementById('intro-screen').classList.remove('hidden');
                }
            } catch (error) {
                console.error("Firebase 에러:", error);
                document.getElementById('loading-screen').innerHTML = "<p class='text-red-600 font-bold'>서버 연결에 실패했습니다. 새로고침 해주세요.</p>";
            }
        }

        // ================= 보안 잠금 (Lock Screen) 시스템 =================
        let currentPinInput = "";
        
        function showLockScreen() {
            document.getElementById('intro-screen').classList.add('hidden');
            document.getElementById('editor-screen').classList.add('hidden');
            document.getElementById('lock-screen').classList.remove('hidden');
            document.getElementById('lock-screen-msg').textContent = "내 생기부를 보호하기 위해 잠금을 해제하세요.";
            document.getElementById('lock-screen-msg').classList.replace('text-red-400', 'text-gray-400');
            
            if (securitySettings.type === 'pin') {
                document.getElementById('lock-pin-ui').classList.remove('hidden');
                document.getElementById('lock-pattern-ui').classList.add('hidden');
                clearPin();
            } else if (securitySettings.type === 'pattern') {
                document.getElementById('lock-pin-ui').classList.add('hidden');
                document.getElementById('lock-pattern-ui').classList.remove('hidden');
                buildPatternGrid('unlock-pattern-container');
            }
        }

        function pressPin(num) {
            if (currentPinInput.length < 4) {
                currentPinInput += num;
                updatePinDots();
                if (currentPinInput.length === 4) {
                    setTimeout(() => verifyLock(currentPinInput), 100);
                }
            }
        }
        function clearPin() {
            currentPinInput = "";
            updatePinDots();
        }
        function updatePinDots() {
            const dots = document.getElementById('pin-dots').children;
            for (let i = 0; i < 4; i++) {
                if (i < currentPinInput.length) {
                    dots[i].classList.replace('border-gray-400', 'border-blue-500');
                    dots[i].classList.add('bg-blue-500');
                } else {
                    dots[i].classList.replace('border-blue-500', 'border-gray-400');
                    dots[i].classList.remove('bg-blue-500');
                }
            }
        }

        async function verifyLock(inputCode) {
            if (inputCode === securitySettings.code) {
                document.getElementById('lock-screen').classList.add('hidden');
                if (pendingTargetGrade) {
                    await startApp(pendingTargetGrade);
                } else {
                    document.getElementById('intro-screen').classList.remove('hidden');
                }
            } else {
                const msgEl = document.getElementById('lock-screen-msg');
                msgEl.textContent = securitySettings.type === 'pin' ? "비밀번호가 일치하지 않습니다." : "패턴이 일치하지 않습니다.";
                msgEl.classList.replace('text-gray-400', 'text-red-400');
                
                const uiBox = securitySettings.type === 'pin' ? document.getElementById('lock-pin-ui') : document.getElementById('lock-pattern-ui');
                uiBox.classList.add('animate-shake');
                setTimeout(() => uiBox.classList.remove('animate-shake'), 400);

                if (securitySettings.type === 'pin') clearPin();
                if (securitySettings.type === 'pattern') resetPatternDots('unlock-pattern-container');
            }
        }

        // ================= 보안 설정(Settings Modal) 시스템 =================
        function openSecuritySettings() {
            document.getElementById('security-modal').classList.remove('hidden');
            document.getElementById('security-modal').classList.add('flex');
            document.getElementById('lock-type-select').value = securitySettings.type || 'none';
            toggleLockSetupUI();
            if(securitySettings.type === 'pin') {
                document.getElementById('setup-pin-input').value = securitySettings.code;
            }
        }

        function closeSecuritySettings() {
            document.getElementById('security-modal').classList.add('hidden');
            document.getElementById('security-modal').classList.remove('flex');
        }

        function toggleLockSetupUI() {
            const type = document.getElementById('lock-type-select').value;
            document.getElementById('setup-pin-ui').classList.add('hidden');
            document.getElementById('setup-pattern-ui').classList.add('hidden');

            if (type === 'pin') {
                document.getElementById('setup-pin-ui').classList.remove('hidden');
                document.getElementById('setup-pin-input').value = '';
            } else if (type === 'pattern') {
                document.getElementById('setup-pattern-ui').classList.remove('hidden');
                buildPatternGrid('setup-pattern-container');
                document.getElementById('setup-pattern-msg').textContent = '';
            }
        }

        async function saveSecuritySettings() {
            const type = document.getElementById('lock-type-select').value;
            let code = '';

            if (type === 'pin') {
                code = document.getElementById('setup-pin-input').value;
                if (code.length !== 4) return showToast("4자리 비밀번호를 정확히 입력해주세요.");
            } else if (type === 'pattern') {
                code = currentPatternCode.join('');
                if (code.length < 4) return showToast("패턴은 최소 4개 이상의 점을 연결해야 합니다.");
            }

            const newSettings = { type: type, code: code };
            securitySettings = newSettings;
            
            if (currentUser) {
                localStorage.setItem(`pocket-sgb-${currentUser.uid}-security`, JSON.stringify(newSettings));
                if(db) {
                    try {
                        await setDoc(doc(db, 'artifacts', appId, 'users', currentUser.uid, 'settings', 'security'), newSettings);
                    } catch(e) { }
                }
            }

            closeSecuritySettings();
            showToast(type === 'none' ? "앱 잠금이 해제되었습니다." : "보안 잠금이 설정되었습니다.");
        }


        // ================= 비밀 패턴 그리기 엔진 =================
        function buildPatternGrid(containerId) {
            const container = document.getElementById(containerId);
            container.innerHTML = '';
            currentPatternCode = [];
            
            for(let i=1; i<=9; i++) {
                const dot = document.createElement('div');
                dot.className = 'pattern-dot w-12 h-12 rounded-full bg-gray-300 transition-colors cursor-pointer flex items-center justify-center relative';
                dot.dataset.val = i;
                container.appendChild(dot);
            }

            container.addEventListener('pointerdown', (e) => startPatternDrawing(e, containerId));
            window.addEventListener('pointermove', (e) => movePatternDrawing(e, containerId)); 
            window.addEventListener('pointerup', () => endPatternDrawing(containerId));
        }

        function resetPatternDots(containerId) {
            const container = document.getElementById(containerId);
            if(!container) return;
            container.querySelectorAll('.pattern-dot').forEach(d => {
                d.classList.remove('bg-blue-500');
                d.classList.add('bg-gray-300');
            });
            currentPatternCode = [];
        }

        function clearSetupPattern() {
            resetPatternDots('setup-pattern-container');
            document.getElementById('setup-pattern-msg').textContent = '';
        }

        function startPatternDrawing(e, containerId) {
            e.preventDefault();
            isDrawingPattern = true;
            activePatternContainerId = containerId;
            resetPatternDots(containerId);
            processPatternPoint(e.clientX, e.clientY);
        }

        function movePatternDrawing(e, targetContainerId) {
            if (!isDrawingPattern || activePatternContainerId !== targetContainerId) return;
            processPatternPoint(e.clientX, e.clientY);
        }

        function processPatternPoint(x, y) {
            const el = document.elementFromPoint(x, y);
            if (el && el.classList.contains('pattern-dot') && el.closest('.pattern-container').id === activePatternContainerId) {
                const val = el.dataset.val;
                if (!currentPatternCode.includes(val)) {
                    currentPatternCode.push(val);
                    el.classList.remove('bg-gray-300');
                    el.classList.add('bg-blue-500'); 
                }
            }
        }

        function endPatternDrawing(containerId) {
            if (!isDrawingPattern || activePatternContainerId !== containerId) return;
            isDrawingPattern = false;
            
            const finalCode = currentPatternCode.join('');
            if(finalCode.length > 0) {
                if (containerId === 'setup-pattern-container') {
                    if (finalCode.length < 4) {
                        document.getElementById('setup-pattern-msg').textContent = "너무 짧습니다 (4개 이상 연결)";
                        document.getElementById('setup-pattern-msg').classList.replace('text-blue-600', 'text-red-500');
                        setTimeout(clearSetupPattern, 1000);
                    } else {
                        document.getElementById('setup-pattern-msg').textContent = "패턴이 기록되었습니다. [저장 적용]을 누르세요.";
                        document.getElementById('setup-pattern-msg').classList.replace('text-red-500', 'text-blue-600');
                    }
                } 
                else if (containerId === 'unlock-pattern-container') {
                    verifyLock(finalCode);
                }
            }
            activePatternContainerId = null;
        }


        // ================= Auth 핸들링 ================= 
        function showError(msg) {
            document.getElementById('error-modal-msg').textContent = msg;
            document.getElementById('error-modal').classList.remove('hidden');
            document.getElementById('error-modal').classList.add('flex');
        }

        function closeErrorModal() {
            document.getElementById('error-modal').classList.add('hidden');
            document.getElementById('error-modal').classList.remove('flex');
        }

        async function handleLogin() {
            const email = document.getElementById('email-input').value;
            const pwd = document.getElementById('pwd-input').value;
            if(!email || !pwd) return showToast("이메일과 비밀번호를 모두 입력해주세요.");
            try {
                document.getElementById('loading-screen').classList.remove('hidden');
                await signInWithEmailAndPassword(auth, email, pwd);
                showToast("로그인 성공! 환영합니다.");
            } catch(e) {
                document.getElementById('loading-screen').classList.add('hidden');
                showError("로그인 실패: 아이디나 비밀번호가 틀렸습니다.\n(또는 아직 가입하지 않은 계정입니다.)");
            }
        }

        async function handleSignup() {
            const email = document.getElementById('email-input').value;
            const pwd = document.getElementById('pwd-input').value;
            if(!email || !pwd) return showToast("사용할 이메일과 비밀번호를 입력해주세요.");
            if(pwd.length < 6) return showToast("비밀번호는 최소 6자 이상이어야 합니다.");
            try {
                document.getElementById('loading-screen').classList.remove('hidden');
                await createUserWithEmailAndPassword(auth, email, pwd);
                showToast("회원가입 완료! 내 생기부 작성을 시작하세요.");
            } catch(e) {
                document.getElementById('loading-screen').classList.add('hidden');
                if (e.code === 'auth/operation-not-allowed') {
                    showError("💡 가입 오류 (설정 필요)\n\n파이어베이스(Firebase) 콘솔에 접속하신 후\n[Authentication] ➡️ [Sign-in method] 탭에서\n'이메일/비밀번호' 로그인을 사용 설정(Enable) 해주세요!");
                } else if (e.code === 'auth/email-already-in-use') {
                    showError("이미 가입된 이메일입니다.\n바로 '로그인' 버튼을 눌러주세요.");
                } else if (e.code === 'auth/invalid-email') {
                    showError("이메일 형식이 올바르지 않습니다.");
                } else {
                    showError("가입 실패: " + e.message);
                }
            }
        }

        async function handleGuestLogin() {
            try {
                document.getElementById('loading-screen').classList.remove('hidden');
                await signInAnonymously(auth);
            } catch(e) {
                document.getElementById('loading-screen').classList.add('hidden');
                showToast("게스트 로그인에 실패했습니다.");
            }
        }

        async function handleLogout() {
            try {
                await signOut(auth);
                currentGrade = null;
                document.getElementById('email-input').value = '';
                document.getElementById('pwd-input').value = '';
                showToast("안전하게 로그아웃 되었습니다.");
            } catch(e) {
                showToast("로그아웃 중 오류가 발생했습니다.");
            }
        }

        // ================= 생기부 기능 로직 ================= 
        async function startApp(grade) {
            currentGrade = grade;
            document.getElementById('intro-screen').classList.add('hidden');
            document.getElementById('editor-screen').classList.remove('hidden');
            document.getElementById('grade-badge').textContent = `${grade}학년`;

            if (currentUser) {
                localStorage.setItem(`pocket-sgb-${currentUser.uid}-last-grade`, grade);
                if (db) {
                    try {
                        const prefRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'settings', 'preferences');
                        await setDoc(prefRef, { lastGrade: grade }, { merge: true });
                    } catch (e) { }
                }
            }

            initSmartTextareas();
            await loadData(grade);
        }

        function resetApp() {
            document.getElementById('confirm-modal').classList.remove('hidden');
            document.getElementById('confirm-modal').classList.add('flex');
        }

        function closeConfirmModal() {
            document.getElementById('confirm-modal').classList.add('hidden');
            document.getElementById('confirm-modal').classList.remove('flex');
        }

        async function executeResetApp() {
            closeConfirmModal();
            await saveData();
            document.getElementById('editor-screen').classList.add('hidden');
            document.getElementById('intro-screen').classList.remove('hidden');
            
            document.getElementById('awards-list').innerHTML = '';
            document.getElementById('volunteer-list').innerHTML = '';
            document.getElementById('subjects-list').innerHTML = '';
            textareasMap.clear();
        }

        function escapeHtml(text) {
            return text.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        }

        function createSmartTextarea(containerId, limit, initialText = "", counterId = null) {
            const container = document.getElementById(containerId);
            if (!container) return;

            container.innerHTML = '';
            const backdrop = document.createElement('div');
            backdrop.className = 'highlight-backdrop';
            const textarea = document.createElement('textarea');
            textarea.className = 'highlight-textarea';
            textarea.placeholder = "내용을 입력하세요...";
            textarea.value = initialText;
            textarea.setAttribute('spellcheck', 'false');

            container.appendChild(backdrop);
            container.appendChild(textarea);

            const counterEl = counterId ? document.getElementById(counterId) : null;

            const updateHighlight = () => {
                let text = textarea.value;
                let renderText = text;
                if (renderText.endsWith('\n')) renderText += ' '; 

                if (text.length <= limit) {
                    backdrop.innerHTML = escapeHtml(renderText);
                } else {
                    const safeUnder = escapeHtml(renderText.substring(0, limit));
                    const safeOver = escapeHtml(renderText.substring(limit));
                    backdrop.innerHTML = safeUnder + '<mark>' + safeOver + '</mark>';
                }

                if (counterEl) {
                    counterEl.textContent = `${text.length} / ${limit}자`;
                    if (text.length > limit) {
                        counterEl.classList.add('text-red-600', 'font-bold');
                        counterEl.classList.remove('text-gray-500');
                    } else {
                        counterEl.classList.add('text-gray-500');
                        counterEl.classList.remove('text-red-600', 'font-bold');
                    }
                }
                textareasMap.set(containerId, text);
            };

            textarea.addEventListener('input', () => {
                updateHighlight();
                debouncedSave();
            });
            
            textarea.addEventListener('scroll', () => {
                backdrop.scrollTop = textarea.scrollTop;
                backdrop.scrollLeft = textarea.scrollLeft;
            });

            updateHighlight();
            return textarea;
        }

        function initSmartTextareas() {
            createSmartTextarea('container-act-1', 500, "", 'counter-act-1');
            createSmartTextarea('container-act-2', 500, "", 'counter-act-2');
            createSmartTextarea('container-act-3', 700, "", 'counter-act-3');
            createSmartTextarea('container-behavior', 500, "", 'counter-behavior');
        }

        function addAward(name = "", date = "", agency = "") {
            const list = document.getElementById('awards-list');
            const id = `award-${Date.now()}`;
            const tr = document.createElement('tr');
            tr.id = id;
            tr.innerHTML = `
                <td class="p-1"><input type="text" placeholder="수상명" value="${name}" class="award-name w-full bg-transparent border border-transparent hover:border-gray-300 focus:border-blue-500 p-1 text-sm outline-none transition-colors"></td>
                <td class="p-1"><input type="text" placeholder="수여기관" value="${agency}" class="award-agency w-full text-center bg-transparent border border-transparent hover:border-gray-300 focus:border-blue-500 p-1 text-sm outline-none transition-colors"></td>
                <td class="p-1"><input type="date" value="${date}" class="award-date w-full text-center bg-transparent border border-transparent hover:border-gray-300 focus:border-blue-500 p-1 text-sm outline-none transition-colors"></td>
                <td class="p-1 text-center no-print"><button onclick="document.getElementById('${id}').remove(); window.debouncedSave();" class="text-xs text-red-600 hover:underline">삭제</button></td>
            `;
            list.appendChild(tr);
            tr.querySelectorAll('input').forEach(input => input.addEventListener('input', debouncedSave));
        }

        function addVolunteer(date = "", place = "", content = "", hours = "") {
            const list = document.getElementById('volunteer-list');
            const id = `vol-${Date.now()}-${Math.floor(Math.random()*1000)}`;
            const tr = document.createElement('tr');
            tr.id = id;
            tr.innerHTML = `
                <td class="p-1"><input type="date" value="${date}" class="vol-date w-full text-center bg-transparent border border-transparent hover:border-gray-300 focus:border-blue-500 p-1 text-sm outline-none transition-colors"></td>
                <td class="p-1"><input type="text" placeholder="장소/기관" value="${place}" class="vol-place w-full text-center bg-transparent border border-transparent hover:border-gray-300 focus:border-blue-500 p-1 text-sm outline-none transition-colors"></td>
                <td class="p-1"><input type="text" placeholder="활동내용" value="${content}" class="vol-content w-full bg-transparent border border-transparent hover:border-gray-300 focus:border-blue-500 p-1 text-sm outline-none transition-colors"></td>
                <td class="p-1"><input type="number" placeholder="시간" value="${hours}" class="vol-hours w-full text-center bg-transparent border border-transparent hover:border-gray-300 focus:border-blue-500 p-1 text-sm outline-none transition-colors"></td>
                <td class="p-1 text-center no-print"><button onclick="document.getElementById('${id}').remove(); window.debouncedSave();" class="text-xs text-red-600 hover:underline">삭제</button></td>
            `;
            list.appendChild(tr);
            tr.querySelectorAll('input').forEach(input => input.addEventListener('input', debouncedSave));
        }

        function addSubject(subjectName = "", content = "") {
            const list = document.getElementById('subjects-list');
            const uniqueId = Date.now() + Math.floor(Math.random() * 1000);
            const containerId = `subject-container-${uniqueId}`;
            const counterId = `subject-counter-${uniqueId}`;
            const wrapId = `subject-wrap-${uniqueId}`;
            const tr = document.createElement('tr');
            tr.id = wrapId;
            tr.className = 'subject-item';
            tr.innerHTML = `
                <td class="bg-gray-50 text-center p-2 align-top">
                    <input type="text" value="${subjectName}" placeholder="과목명" class="subject-name w-full text-center font-bold bg-transparent border-b border-gray-300 focus:border-blue-500 p-1 outline-none mb-1">
                    <div class="text-xs text-gray-500 mt-2 no-print" id="${counterId}">0 / 500자</div>
                </td>
                <td class="p-0 relative align-top">
                    <div id="${containerId}" class="smart-textarea-container" data-limit="500"></div>
                </td>
                <td class="p-2 text-center align-middle no-print">
                    <button onclick="removeSubject('${wrapId}')" class="text-xs text-red-600 hover:underline">삭제</button>
                </td>
            `;
            list.appendChild(tr);
            createSmartTextarea(containerId, 500, content, counterId);
            tr.querySelector('.subject-name').addEventListener('input', debouncedSave);
        }

        function removeSubject(wrapId) {
            document.getElementById('delete-modal').classList.remove('hidden');
            document.getElementById('delete-modal').classList.add('flex');
            document.getElementById('confirm-delete-btn').onclick = () => {
                document.getElementById(wrapId).remove();
                debouncedSave();
                closeDeleteModal();
            };
        }

        function closeDeleteModal() {
            document.getElementById('delete-modal').classList.add('hidden');
            document.getElementById('delete-modal').classList.remove('flex');
        }

        async function saveData() {
            if (!currentGrade || !currentUser) return;
            const data = {
                awards: [], volunteers: [],
                activities: {
                    act1: textareasMap.get('container-act-1') || "",
                    act2: textareasMap.get('container-act-2') || "",
                    act3: textareasMap.get('container-act-3') || ""
                },
                subjects: [], behavior: textareasMap.get('container-behavior') || ""
            };
            document.querySelectorAll('#awards-list > tr').forEach(tr => {
                data.awards.push({
                    name: tr.querySelector('.award-name').value,
                    agency: tr.querySelector('.award-agency').value,
                    date: tr.querySelector('.award-date').value
                });
            });
            document.querySelectorAll('#volunteer-list > tr').forEach(tr => {
                data.volunteers.push({
                    date: tr.querySelector('.vol-date').value,
                    place: tr.querySelector('.vol-place').value,
                    content: tr.querySelector('.vol-content').value,
                    hours: tr.querySelector('.vol-hours').value
                });
            });
            document.querySelectorAll('.subject-item').forEach(tr => {
                const name = tr.querySelector('.subject-name').value;
                const containerId = tr.querySelector('.smart-textarea-container').id;
                const content = textareasMap.get(containerId) || "";
                data.subjects.push({ name, content });
            });
            try {
                if(db) {
                    const docRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'sgb_grades', String(currentGrade));
                    await setDoc(docRef, data);
                    showToast("☁️ 클라우드에 안전하게 저장되었습니다.");
                }
            } catch (e) {
                console.error("저장 에러:", e);
                localStorage.setItem(`pocket-sgb-${currentUser.uid}-grade-${currentGrade}`, JSON.stringify(data));
            }
        }

        let debounceTimer;
        function debouncedSave() {
            clearTimeout(debounceTimer);
            debounceTimer = setTimeout(saveData, 1500); 
        }

        async function loadData(grade) {
            document.getElementById('awards-list').innerHTML = '';
            document.getElementById('volunteer-list').innerHTML = '';
            document.getElementById('subjects-list').innerHTML = '';
            let data = null;
            if (db && currentUser) {
                try {
                    const docRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'sgb_grades', String(grade));
                    const docSnap = await getDoc(docRef);
                    if (docSnap.exists()) data = docSnap.data();
                } catch (e) {
                    const localData = localStorage.getItem(`pocket-sgb-${currentUser.uid}-grade-${grade}`);
                    if(localData) data = JSON.parse(localData);
                }
            }
            if (data) {
                if(data.awards) data.awards.forEach(award => addAward(award.name, award.date, award.agency));
                if(data.volunteers) data.volunteers.forEach(vol => addVolunteer(vol.date, vol.place, vol.content, vol.hours));
                createSmartTextarea('container-act-1', 500, data.activities.act1, 'counter-act-1');
                createSmartTextarea('container-act-2', 500, data.activities.act2, 'counter-act-2');
                createSmartTextarea('container-act-3', 700, data.activities.act3, 'counter-act-3');
                if (data.subjects && data.subjects.length > 0) data.subjects.forEach(sub => addSubject(sub.name, sub.content));
                else defaultSubjects[grade].forEach(subjectName => addSubject(subjectName, ""));
                createSmartTextarea('container-behavior', 500, data.behavior, 'counter-behavior');
            } else {
                createSmartTextarea('container-act-1', 500, "", 'counter-act-1');
                createSmartTextarea('container-act-2', 500, "", 'counter-act-2');
                createSmartTextarea('container-act-3', 700, "", 'counter-act-3');
                createSmartTextarea('container-behavior', 500, "", 'counter-behavior');
                defaultSubjects[grade].forEach(subjectName => addSubject(subjectName, ""));
            }
        }

        function showToast(msg) {
            const toast = document.getElementById('toast');
            toast.textContent = msg;
            toast.classList.remove('opacity-0');
            setTimeout(() => toast.classList.add('opacity-0'), 2000);
        }

        function exportToDocument() {
            const contentClone = document.getElementById('document-content').cloneNode(true);
            contentClone.style.maxWidth = '100%';
            contentClone.style.width = '100%';
            contentClone.querySelectorAll('.no-print').forEach(el => el.remove());
            contentClone.querySelectorAll('colgroup').forEach(cg => cg.remove());
            const originalInputs = document.getElementById('document-content').querySelectorAll('input');
            const clonedInputs = contentClone.querySelectorAll('input');
            originalInputs.forEach((input, index) => {
                const span = document.createElement('span');
                span.innerHTML = input.value ? escapeHtml(input.value) : '&nbsp;&nbsp;&nbsp;&nbsp;';
                clonedInputs[index].parentNode.replaceChild(span, clonedInputs[index]);
            });
            contentClone.querySelectorAll('.smart-textarea-container').forEach(container => {
                const text = textareasMap.get(container.id) || "";
                const div = document.createElement('div');
                div.style.padding = '8px';
                div.style.lineHeight = '1.6';
                div.style.whiteSpace = 'pre-wrap';
                div.innerHTML = escapeHtml(text).replace(/\n/g, '<br>');
                container.parentNode.replaceChild(div, container);
            });
            contentClone.querySelectorAll('table').forEach(table => {
                table.style.borderCollapse = 'collapse';
                table.style.width = '100%';
                table.setAttribute('width', '100%');
                table.style.marginBottom = '20px';
                table.style.border = '1px solid black';
            });
            contentClone.querySelectorAll('th, td').forEach(cell => {
                cell.style.border = '1px solid black';
                cell.style.padding = '8px';
            });
            const blob = new Blob(['\ufeff', contentClone.innerHTML], { type: 'application/msword' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `학교생활기록부_${currentGrade}학년.doc`;
            a.click();
            URL.revokeObjectURL(url);
        }

        window.handleLogin = handleLogin; window.handleSignup = handleSignup; window.handleGuestLogin = handleGuestLogin; window.handleLogout = handleLogout;
        window.openSecuritySettings = openSecuritySettings; window.closeSecuritySettings = closeSecuritySettings; window.toggleLockSetupUI = toggleLockSetupUI;
        window.saveSecuritySettings = saveSecuritySettings; window.pressPin = pressPin; window.clearPin = clearPin; window.clearSetupPattern = clearSetupPattern;
        window.startApp = startApp; window.resetApp = resetApp; window.executeResetApp = executeResetApp; window.closeConfirmModal = closeConfirmModal;
        window.addAward = addAward; window.addVolunteer = addVolunteer; window.addSubject = addSubject; window.removeSubject = removeSubject; window.closeDeleteModal = closeDeleteModal;
        window.saveData = saveData; window.debouncedSave = debouncedSave; window.exportToDocument = exportToDocument; window.closeErrorModal = closeErrorModal;
        window.onload = initFirebase;
    </script>
</body>
</html>
