<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>DUL 만화 뷰어</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
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
            overflow: hidden;
        }
        .fade-transition {
            transition: opacity 0.2s ease-in-out;
        }
    </style>
</head>
<body class="h-screen w-screen flex flex-col justify-between overflow-hidden">

    <!-- 상단 컨트롤바 -->
    <header id="top-bar" class="absolute top-0 left-0 right-0 z-50 bg-slate-900/80 backdrop-blur-md border-b border-slate-800 px-4 py-3 flex items-center justify-between transition-transform duration-300">
        <div class="flex items-center space-x-3">
            <span class="text-sm font-semibold tracking-wider text-slate-300">DUL Viewer</span>
        </div>
        <div class="flex items-center space-x-2">
            <button onclick="changePage(-1)" class="w-9 h-9 rounded-full bg-slate-800 hover:bg-slate-700 flex items-center justify-center text-slate-200 active:scale-95 transition">
                <i class="fa-solid fa-chevron-left text-xs"></i>
            </button>
            <div class="flex items-center space-x-1 bg-slate-800 px-3 py-1.5 rounded-lg border border-slate-700">
                <input type="number" id="page-input" min="81" max="278" value="81" 
                    class="w-12 text-center bg-transparent text-white font-bold focus:outline-none text-sm"
                    onchange="jumpToPage(this.value)" onkeydown="if(event.key==='Enter') jumpToPage(this.value)">
                <span class="text-slate-400 text-sm">/ 278</span>
            </div>
            <button onclick="changePage(1)" class="w-9 h-9 rounded-full bg-slate-800 hover:bg-slate-700 flex items-center justify-center text-slate-200 active:scale-95 transition">
                <i class="fa-solid fa-chevron-right text-xs"></i>
            </button>
        </div>
        <button onclick="toggleUI()" class="w-9 h-9 rounded-full bg-slate-800 hover:bg-slate-700 flex items-center justify-center text-slate-200 transition">
            <i class="fa-solid fa-eye text-sm" id="ui-icon"></i>
        </button>
    </header>

    <!-- 중앙 이미지 뷰어 영역 -->
    <main class="flex-1 w-full h-full flex items-center justify-center relative overflow-hidden" onclick="handleScreenClick(event)">
        <!-- 로딩 스피너 -->
        <div id="loading" class="absolute inset-0 flex items-center justify-center bg-slate-950/60 z-10 hidden">
            <div class="w-8 h-8 border-4 border-indigo-500 border-t-transparent rounded-full animate-spin"></div>
        </div>
        
        <!-- 만화 이미지 -->
        <img id="manga-image" src="" alt="만화 페이지" 
             class="max-h-full max-w-full object-contain cursor-pointer fade-transition"
             onload="hideLoading()" onerror="handleImageError()">
             
        <!-- 터치 안내 오버레이 (첫 방문용 가이드) -->
        <div id="guide-overlay" class="absolute inset-x-0 bottom-12 flex justify-center pointer-events-none opacity-60 transition-opacity duration-500">
            <div class="bg-black/60 text-slate-300 text-xs px-3 py-1.5 rounded-full backdrop-blur-sm">
                화면 좌우를 터치하여 넘기세요
            </div>
        </div>
    </main>

    <!-- 하단 컨트롤바 -->
    <footer id="bottom-bar" class="absolute bottom-0 left-0 right-0 z-50 bg-slate-900/80 backdrop-blur-md border-t border-slate-800 px-6 py-3 flex flex-col space-y-2 transition-transform duration-300">
        <input type="range" id="page-slider" min="81" max="278" value="81" 
            class="w-full accent-indigo-500 cursor-pointer"
            oninput="onSliderInput(this.value)" onchange="jumpToPage(this.value)">
        <div class="flex justify-between text-xs text-slate-400 font-medium">
            <span>81 (시작)</span>
            <span id="current-progress">81 / 278</span>
            <span>278 (끝)</span>
        </div>
    </footer>

    <script>
        let currentPage = 81;
        const minPage = 81;
        const maxPage = 278;
        let uiVisible = true;

        const imageElement = document.getElementById('manga-image');
        const pageInput = document.getElementById('page-input');
        const pageSlider = document.getElementById('page-slider');
        const loadingElement = document.getElementById('loading');
        const progressText = document.getElementById('current-progress');
        const topBar = document.getElementById('top-bar');
        const bottomBar = document.getElementById('bottom-bar');
        const guideOverlay = document.getElementById('guide-overlay');

        function loadPage(page) {
            if (page < minPage) page = minPage;
            if (page > maxPage) page = maxPage;
            currentPage = page;

            pageInput.value = currentPage;
            pageSlider.value = currentPage;
            progressText.innerText = `${currentPage} / ${maxPage}`;

            loadingElement.classList.remove('hidden');
            imageElement.style.opacity = '0.3';
            
            // R2 스토리지 주소 조합
            imageElement.src = `https://pub-8eae9ddb98e3498bbee34d7d8c6230d9.r2.dev/DUL/${currentPage}.webp`;

            // 3초 후 가이드 숨기기
            if (currentPage > 82) {
                guideOverlay.classList.add('opacity-0');
            }
        }

        function hideLoading() {
            loadingElement.classList.add('hidden');
            imageElement.style.opacity = '1';
        }

        function handleImageError() {
            loadingElement.classList.add('hidden');
            imageElement.style.opacity = '1';
            // 이미지가 없을 경우 대체 처리 혹은 알림
        }

        function changePage(direction) {
            let next = currentPage + direction;
            if (next >= minPage && next <= maxPage) {
                loadPage(next);
            }
        }

        function jumpToPage(val) {
            let page = parseInt(val);
            if (isNaN(page)) return;
            loadPage(page);
        }

        function onSliderInput(val) {
            progressText.innerText = `${val} / ${maxPage}`;
        }

        function handleScreenClick(event) {
            const screenWidth = window.innerWidth;
            const clickX = event.clientX;

            // 화면 왼쪽 30% 터치하면 이전 페이지, 오른쪽 70% 터치하면 다음 페이지
            if (clickX < screenWidth * 0.3) {
                changePage(-1);
            } else if (clickX > screenWidth * 0.7) {
                changePage(1);
            } else {
                // 가운데를 누르면 상하단 메뉴 토글
                toggleUI();
            }
        }

        function toggleUI() {
            uiVisible = !uiVisible;
            if (uiVisible) {
                topBar.classList.remove('-translate-y-full');
                bottomBar.classList.remove('translate-y-full');
            } else {
                topBar.classList.add('-translate-y-full');
                bottomBar.classList.add('translate-y-full');
            }
        }

        // 키보드 방향키 조작 지원
        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft' || e.key === 'ArrowUp') {
                changePage(-1);
            } else if (e.key === 'ArrowRight' || e.key === 'ArrowDown' || e.key === ' ') {
                changePage(1);
            }
        });

        // 초기 로드
        loadPage(currentPage);
    </script>
</body>
</html>
