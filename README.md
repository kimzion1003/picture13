```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>웹 만화 뷰어 (R2 Viewer)</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body {
            background-color: #0f172a;
            color: #f8fafc;
            font-family: 'Inter', sans-serif;
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }
        /* Custom scrollbar for clean UI */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #1e293b;
        }
        ::-webkit-scrollbar-thumb {
            background: #475569;
            border-radius: 3px;
        }
        .fade-transition {
            transition: opacity 0.2s ease-in-out;
        }
    </style>
</head>
<body class="h-screen flex flex-col overflow-hidden">

    <!-- 상단 네비게이션바 (자동 숨김 가능 또는 상단 고정) -->
    <header id="top-bar" class="bg-slate-900/90 backdrop-blur-md border-b border-slate-800 px-4 py-3 flex items-center justify-between z-20 transition-transform duration-300">
        <div class="flex items-center space-x-3">
            <span class="text-sm font-semibold tracking-wider text-slate-300">DUL VIEWER</span>
            <span class="bg-indigo-600/30 text-indigo-400 text-xs px-2 py-0.5 rounded-full font-medium">Range: 81 - 278</span>
        </div>
        
        <!-- 페이지 직접 이동 및 슬라이더 컨트롤 -->
        <div class="flex items-center space-x-2">
            <button id="btn-prev" class="bg-slate-800 hover:bg-slate-700 active:bg-slate-600 text-slate-200 px-3 py-1.5 rounded-lg text-sm transition flex items-center shadow">
                <i class="fa-solid fa-chevron-left mr-1"></i> <span class="hidden sm:inline">이전</span>
            </button>
            
            <div class="flex items-center bg-slate-800 px-2 py-1 rounded-lg border border-slate-700">
                <input type="number" id="page-input" min="81" max="278" value="81" 
                    class="w-14 bg-transparent text-center font-bold text-indigo-400 focus:outline-none text-sm">
                <span class="text-slate-400 text-sm mx-1">/ 278</span>
            </div>

            <button id="btn-next" class="bg-indigo-600 hover:bg-indigo-500 active:bg-indigo-700 text-white px-3 py-1.5 rounded-lg text-sm transition flex items-center shadow">
                <span class="hidden sm:inline">다음</span> <i class="fa-solid fa-chevron-right ml-1"></i>
            </button>
        </div>

        <div class="flex items-center space-x-2">
            <button id="btn-toggle-ui" class="text-slate-400 hover:text-white p-2 rounded-lg transition" title="인터페이스 숨기기/보기">
                <i class="fa-solid fa-eye-slash text-lg"></i>
            </button>
        </div>
    </header>

    <!-- 메인 이미지 뷰어 영역 (화면 터치 시 다음 장 이동 지원) -->
    <main id="viewer-container" class="relative flex-1 flex items-center justify-center overflow-auto bg-slate-950 p-2 sm:p-4 cursor-pointer">
        
        <!-- 로딩 스피너 및 상태 메시지 -->
        <div id="loading-spinner" class="absolute inset-0 flex flex-col items-center justify-center bg-slate-950/80 z-10 pointer-events-none transition-opacity duration-200">
            <div class="w-12 h-12 border-4 border-indigo-500 border-t-transparent rounded-full animate-spin mb-3"></div>
            <p id="loading-text" class="text-sm text-slate-300 font-medium">이미지를 불러오는 중... (81)</p>
        </div>

        <!-- 에러 메시지 오버레이 -->
        <div id="error-message" class="absolute hidden flex flex-col items-center justify-center bg-slate-900/95 p-6 rounded-2xl border border-slate-800 z-25 max-w-sm text-center shadow-2xl">
            <i class="fa-solid fa-triangle-exclamation text-amber-400 text-3xl mb-3"></i>
            <h3 class="text-base font-bold text-slate-100 mb-1">이미지를 불러올 수 없습니다</h3>
            <p id="error-detail" class="text-xs text-slate-400 mb-4">해당 번호의 이미지 파일이 존재하지 않거나 네트워크 연결을 확인해주세요.</p>
            <button id="btn-retry" class="bg-indigo-600 hover:bg-indigo-500 text-white text-xs px-4 py-2 rounded-xl font-medium transition">
                다시 시도
            </button>
        </div>

        <!-- 이미지 엘리먼트 -->
        <div class="relative max-w-full max-h-full flex items-center justify-center">
            <img id="manga-image" src="" alt="만화 페이지" class="max-h-[calc(100vh-6rem)] max-w-full object-contain rounded shadow-2xl transition-opacity duration-150 opacity-0">
        </div>

        <!-- 터치 가이드 힌트 (최초 1회 또는 토글 가능) -->
        <div id="touch-guide" class="absolute bottom-6 bg-slate-900/80 backdrop-blur border border-slate-700/50 text-slate-300 text-xs px-4 py-2 rounded-full pointer-events-none transition-opacity duration-500 shadow-lg">
            <i class="fa-solid fa-hand-pointer mr-1.5 text-indigo-400"></i> 화면 좌우를 터치하거나 방향키로 페이지를 넘기세요
        </div>
    </main>

    <!-- 하단 빠른 슬라이더 바 -->
    <footer id="bottom-bar" class="bg-slate-900/90 backdrop-blur-md border-t border-slate-800 px-4 py-3 z-20 flex flex-col space-y-2 transition-transform duration-300">
        <div class="flex items-center justify-between text-xs text-slate-400">
            <span id="current-code-label">상황 코드: 81</span>
            <span id="progress-percent-label">0%</span>
        </div>
        <input type="range" id="page-slider" min="81" max="278" value="81" 
            class="w-full accent-indigo-500 bg-slate-800 h-2 rounded-lg cursor-pointer">
    </footer>

    <!-- 자바스크립트 로직 -->
    <script>
        // 설정 및 상태 관리
        const MIN_PAGE = 81;
        const MAX_PAGE = 278;
        let currentPage = 81;
        let isUiVisible = true;
        let touchStartX = 0;
        let touchEndX = 0;

        // DOM 요소 캐싱
        const mangaImage = document.getElementById('manga-image');
        const pageInput = document.getElementById('page-input');
        const pageSlider = document.getElementById('page-slider');
        const btnPrev = document.getElementById('btn-prev');
        const btnNext = document.getElementById('btn-next');
        const loadingSpinner = document.getElementById('loading-spinner');
        const loadingText = document.getElementById('loading-text');
        const errorMessage = document.getElementById('error-message');
        const errorDetail = document.getElementById('error-detail');
        const btnRetry = document.getElementById('btn-retry');
        const topBar = document.getElementById('top-bar');
        const bottomBar = document.getElementById('bottom-bar');
        const btnToggleUi = document.getElementById('btn-toggle-ui');
        const currentCodeLabel = document.getElementById('current-code-label');
        const progressPercentLabel = document.getElementById('progress-percent-label');
        const touchGuide = document.getElementById('touch-guide');
        const viewerContainer = document.getElementById('viewer-container');

        // R2 이미지 URL 생성 함수
        function getImageUrl(pageNum) {
            return `https://pub-8eae9ddb98e3498bbee34d7d8c6230d9.r2.dev/DUL/${pageNum}.webp`;
        }

        // 페이지 로드 함수
        function loadPage(pageNum) {
            // 범위 검증
            if (pageNum < MIN_PAGE) pageNum = MIN_PAGE;
            if (pageNum > MAX_PAGE) pageNum = MAX_PAGE;

            currentPage = pageNum;

            // UI 컨트롤 업데이트
            pageInput.value = currentPage;
            pageSlider.value = currentPage;
            currentCodeLabel.textContent = `상황 코드: ${currentPage}`;
            
            const progress = Math.round(((currentPage - MIN_PAGE) / (MAX_PAGE - MIN_PAGE)) * 100);
            progressPercentLabel.textContent = `${progress}%`;

            // 상태 표시 초기화
            loadingSpinner.style.opacity = '1';
            loadingSpinner.classList.remove('hidden');
            errorMessage.classList.add('hidden');
            mangaImage.style.opacity = '0';
            loadingText.textContent = `이미지를 불러오는 중... (${currentPage})`;

            // 이미지 소스 설정 및 프리로드
            const targetUrl = getImageUrl(currentPage);
            
            const tempImg = new Image();
            tempImg.onload = function() {
                mangaImage.src = targetUrl;
                loadingSpinner.style.opacity = '0';
                setTimeout(() => {
                    loadingSpinner.classList.add('hidden');
                    mangaImage.style.opacity = '1';
                }, 150);
            };

            tempImg.onerror = function() {
                loadingSpinner.style.opacity = '0';
                loadingSpinner.classList.add('hidden');
                errorDetail.textContent = `상황 코드 ${currentPage}번 이미지(${currentPage}.webp)를 불러오지 못했습니다.`;
                errorMessage.classList.remove('hidden');
            };

            tempImg.src = targetUrl;

            // URL Hash 업데이트 (새로고침 시 현재 페이지 유지)
            window.location.hash = currentPage;
        }

        // 다음 페이지로 이동
        function nextPage() {
            if (currentPage < MAX_PAGE) {
                loadPage(currentPage + 1);
            }
        }

        // 이전 페이지로 이동
        function prevPage() {
            if (currentPage > MIN_PAGE) {
                loadPage(currentPage - 1);
            }
        }

        // UI 토글 (몰입형 전체화면 보기)
        function toggleUI() {
            isUiVisible = !isUiVisible;
            if (isUiVisible) {
                topBar.classList.remove('-translate-y-full');
                bottomBar.classList.remove('translate-y-full');
                btnToggleUi.innerHTML = '<i class="fa-solid fa-eye-slash text-lg"></i>';
            } else {
                topBar.classList.add('-translate-y-full');
                bottomBar.classList.add('translate-y-full');
                btnToggleUi.innerHTML = '<i class="fa-solid fa-eye text-lg"></i>';
            }
        }

        // 이벤트 리스너 설정
        btnNext.addEventListener('click', (e) => {
            e.stopPropagation();
            nextPage();
        });

        btnPrev.addEventListener('click', (e) => {
            e.stopPropagation();
            prevPage();
        });

        btnRetry.addEventListener('click', () => {
            loadPage(currentPage);
        });

        btnToggleUi.addEventListener('click', (e) => {
            e.stopPropagation();
            toggleUI();
        });

        // 직접 번호 입력창 이벤트
        pageInput.addEventListener('change', (e) => {
            const val = parseInt(e.target.value);
            if (!isNaN(val)) {
                loadPage(val);
            } else {
                pageInput.value = currentPage;
            }
        });

        pageInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                const val = parseInt(e.target.value);
                if (!isNaN(val)) {
                    loadPage(val);
                    pageInput.blur();
                }
            }
        });

        // 슬라이더 이벤트
        pageSlider.addEventListener('input', (e) => {
            const val = parseInt(e.target.value);
            pageInput.value = val;
            currentCodeLabel.textContent = `상황 코드: ${val}`;
        });

        pageSlider.addEventListener('change', (e) => {
            const val = parseInt(e.target.value);
            loadPage(val);
        });

        // 키보드 방향키 조작 지원
        window.addEventListener('keydown', (e) => {
            if (e.target.tagName === 'INPUT') return; // 입력 중일 때는 무시
            
            if (e.key === 'ArrowRight' || e.key === 'PageDown' || e.key === ' ') {
                e.preventDefault();
                nextPage();
            } else if (e.key === 'ArrowLeft' || e.key === 'PageUp') {
                e.preventDefault();
                prevPage();
            } else if (e.key === 'h' || e.key === 'H') {
                toggleUI();
            }
        });

        // 화면 터치/클릭 이벤트 (좌우 영역 분할 또는 전체 터치 시 다음 장 이동)
        viewerContainer.addEventListener('click', (e) => {
            // 컨트롤러 내부 클릭이 아닐 경우에만 동작
            const rect = viewerContainer.getBoundingClientRect();
            const clickX = e.clientX - rect.left;
            const containerWidth = rect.width;

            // 터치 가이드 서서히 숨기기
            if (touchGuide) {
                touchGuide.style.opacity = '0';
                setTimeout(() => touchGuide.remove(), 500);
            }

            // 화면 왼쪽 1/3을 누르면 이전, 오른쪽 2/3를 누르면 다음으로 설정 (만화 읽는 방향 고려)
            if (clickX < containerWidth * 0.3) {
                prevPage();
            } else {
                nextPage();
            }
        });

        // 모바일 스와이프 제스처 지원
        viewerContainer.addEventListener('touchstart', (e) => {
            touchStartX = e.changedTouches[0].screenX;
        }, {passive: true});

        viewerContainer.addEventListener('touchend', (e) => {
            touchEndX = e.changedTouches[0].screenX;
            handleSwipe();
        }, {passive: true});

        function handleSwipe() {
            const swipeThreshold = 50;
            if (touchEndX < touchStartX - swipeThreshold) {
                // 왼쪽으로 스와이프 -> 다음 페이지
                nextPage();
            } else if (touchEndX > touchStartX + swipeThreshold) {
                // 오른쪽으로 스와이프 -> 이전 페이지
                prevPage();
            }
        }

        // 초기화: 해시값이 있으면 해당 페이지로 시작
        window.addEventListener('DOMContentLoaded', () => {
            const hash = parseInt(window.location.hash.replace('#', ''));
            if (!isNaN(hash) && hash >= MIN_PAGE && hash <= MAX_PAGE) {
                loadPage(hash);
            } else {
                loadPage(MIN_PAGE);
            }
        });
    </script>
</body>
</html>
```
