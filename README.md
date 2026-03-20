<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>학교알리미</title>
    <style>
        body { font-family: 'Malgun Gothic', sans-serif; margin: 0; padding: 0; background-color: #f4f7f6; padding-bottom: 70px; }
        header { background-color: #4CAF50; color: white; padding: 15px; text-align: center; display: flex; justify-content: space-between; align-items: center; }
        header h1 { margin: 0; font-size: 24px; }
        .login-btn { background-color: white; color: #4CAF50; border: none; padding: 8px 12px; border-radius: 5px; cursor: pointer; font-weight: bold; }
        
        /* 오늘의 날짜 표시 */
        .today-banner { text-align: center; background-color: #e8f5e9; padding: 15px; margin: 10px auto; max-width: 600px; border-radius: 10px; color: #2e7d32; font-size: 18px; font-weight: bold; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }

        .calendar-container { max-width: 600px; margin: 10px auto; background: white; padding: 20px; border-radius: 10px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); }
        .month-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
        .month-header button { background: none; border: none; font-size: 20px; cursor: pointer; }
        .days-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 5px; text-align: center; }
        .day-name { font-weight: bold; color: #555; padding: 10px 0; }
        .day { padding: 15px 0; background-color: #f9f9f9; border-radius: 5px; cursor: pointer; transition: 0.2s; }
        .day:hover { background-color: #e0f2f1; }
        .day.today { background-color: #4CAF50; color: white; font-weight: bold; }
        
        /* 하단 고정 메뉴 (시간표, 급식표, 가정통신문) */
        .bottom-nav { position: fixed; bottom: 0; left: 0; width: 100%; background: #fff; display: flex; box-shadow: 0 -2px 10px rgba(0,0,0,0.1); z-index: 100; }
        .bottom-nav button { flex: 1; padding: 15px 0; border: none; background: none; font-size: 16px; font-weight: bold; color: #555; cursor: pointer; border-right: 1px solid #eee; }
        .bottom-nav button:last-child { border-right: none; }
        .bottom-nav button:hover { background: #f0f8ff; color: #2196F3; }

        /* Modal Styles */
        .modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); overflow-y: auto; z-index: 200; }
        .modal-content { background: white; margin: 5% auto; padding: 20px; width: 90%; max-width: 600px; border-radius: 10px; position: relative; min-height: 50vh; }
        .close-btn { position: absolute; right: 20px; top: 15px; font-size: 28px; cursor: pointer; color: #aaa; }
        .close-btn:hover { color: black; }
        
        .section { margin-top: 20px; padding-top: 15px; }
        .section h3 { margin-top: 0; color: #333; border-bottom: 2px solid #eee; padding-bottom: 5px; }
        .admin-only { display: none; margin-bottom: 15px; background: #f0f8ff; padding: 10px; border-radius: 5px; }
        input[type="text"], input[type="file"] { width: 100%; margin-bottom: 10px; padding: 8px; box-sizing: border-box; border: 1px solid #ccc; border-radius: 4px; }
        button.add-btn { background-color: #2196F3; color: white; border: none; padding: 8px 15px; border-radius: 5px; cursor: pointer; font-weight: bold; }
        button.delete-btn { background-color: #f44336; color: white; border: none; padding: 5px 10px; border-radius: 3px; cursor: pointer; margin-top: 5px; font-size: 12px; display: none; }
        
        .item-card { background: #fafafa; border: 1px solid #ddd; padding: 15px; margin-bottom: 10px; border-radius: 5px; }
        .img-preview { display: flex; gap: 5px; flex-wrap: wrap; margin-top: 10px; }
        .img-preview img { height: 120px; border-radius: 5px; border: 1px solid #ccc; object-fit: cover; }
    </style>
</head>
<body>

<header>
    <div></div>
    <h1>학교알리미</h1>
    <button class="login-btn" id="loginBtn" onclick="toggleLogin()">반장 로그인</button>
</header>

<div class="today-banner" id="todayBanner">
    </div>

<div class="calendar-container">
    <div class="month-header">
        <button onclick="changeMonth(-1)">◀</button>
        <h2 id="monthTitle"></h2>
        <button onclick="changeMonth(1)">▶</button>
    </div>
    <div class="days-grid" id="calendarGrid">
        <div class="day-name">일</div><div class="day-name">월</div><div class="day-name">화</div>
        <div class="day-name">수</div><div class="day-name">목</div><div class="day-name">금</div><div class="day-name">토</div>
    </div>
</div>

<div class="bottom-nav">
    <button onclick="openGlobalModal('timetable', '시간표')">📅 시간표</button>
    <button onclick="openGlobalModal('meal', '급식표')">🍱 급식표</button>
    <button onclick="openGlobalModal('notice', '가정통신문')">📢 가정통신문</button>
</div>

<div id="mainModal" class="modal">
    <div class="modal-content">
        <span class="close-btn" onclick="closeModal()">&times;</span>
        <h2 id="modalTitle" style="color: #4CAF50; margin-top: 0;"></h2>
        <div id="sectionsContainer">
            </div>
    </div>
</div>

<script>
    let isPresident = false;
    let currentDate = new Date();
    const todayDate = new Date(); // 실제 오늘 날짜 고정용
    
    // 현재 모달 상태 저장 (날짜인지, 글로벌 메뉴인지 구분)
    let currentModalType = ""; 
    let currentGlobalCategory = "";

    // 날짜별 데이터 저장소
    let dailyData = JSON.parse(localStorage.getItem('alarmiDaily')) || {};
    // 공통(시간표, 급식표, 가정통신문) 데이터 저장소
    let globalData = JSON.parse(localStorage.getItem('alarmiGlobal')) || { timetable: [], meal: [], notice: [] };

    // 날짜별로 보여줄 항목
    const dailyCategories = [
        { id: 'schedule', name: '오늘의 일정' },
        { id: 'class', name: '수업 내용' }
    ];

    // 1. 오늘의 날짜 상단에 표시
    function showTodayDate() {
        const days = ['일', '월', '화', '수', '목', '금', '토'];
        const year = todayDate.getFullYear();
        const month = todayDate.getMonth() + 1;
        const date = todayDate.getDate();
        const day = days[todayDate.getDay()];
        document.getElementById('todayBanner').innerText = `오늘은 ${year}년 ${month}월 ${date}일 (${day}) 입니다!`;
    }

    // 2. 반장 로그인 로직 (비밀번호: 3038)
    function toggleLogin() {
        if (isPresident) {
            isPresident = false;
            document.getElementById('loginBtn').innerText = "반장 로그인";
            alert("로그아웃 되었습니다.");
            if(document.getElementById('mainModal').style.display === "block"){
                refreshModal();
            }
            return;
        }

        const pwd = prompt("반장 비밀번호를 입력하세요:");
        if (pwd === "3038") {
            isPresident = true;
            document.getElementById('loginBtn').innerText = "로그아웃 (반장)";
            alert("반장으로 로그인되었습니다. 내용을 자유롭게 추가/삭제할 수 있습니다.");
            if(document.getElementById('mainModal').style.display === "block"){
                refreshModal();
            }
        } else if (pwd !== null) {
            alert("비밀번호가 일치하지 않습니다.");
        }
    }

    // 3. 달력 그리기
    function renderCalendar() {
        const year = currentDate.getFullYear();
        const month = currentDate.getMonth();
        document.getElementById('monthTitle').innerText = `${year}년 ${month + 1}월`;
        
        const grid = document.getElementById('calendarGrid');
        while (grid.children.length > 7) { grid.removeChild(grid.lastChild); }

        const firstDay = new Date(year, month, 1).getDay();
        const daysInMonth = new Date(year, month + 1, 0).getDate();

        for (let i = 0; i < firstDay; i++) {
            grid.appendChild(document.createElement('div'));
        }

        for (let i = 1; i <= daysInMonth; i++) {
            let dayDiv = document.createElement('div');
            dayDiv.className = 'day';
            dayDiv.innerText = i;
            
            if (year === todayDate.getFullYear() && month === todayDate.getMonth() && i === todayDate.getDate()) {
                dayDiv.classList.add('today');
            }

            dayDiv.onclick = () => openDailyModal(year, month + 1, i);
            grid.appendChild(dayDiv);
        }
    }

    function changeMonth(step) {
        currentDate.setMonth(currentDate.getMonth() + step);
        renderCalendar();
    }

    // 4. 날짜 클릭 시 열리는 모달 (일정, 수업내용)
    function openDailyModal(year, month, day) {
        currentModalType = "daily";
        currentGlobalCategory = `${year}-${String(month).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
        
        document.getElementById('modalTitle').innerText = `${month}월 ${day}일 기록`;
        document.getElementById('mainModal').style.display = "block";
        refreshModal();
    }

    // 5. 하단 버튼 클릭 시 열리는 모달 (시간표, 급식표, 가정통신문)
    function openGlobalModal(categoryId, title) {
        currentModalType = "global";
        currentGlobalCategory = categoryId;

        document.getElementById('modalTitle').innerText = title;
        document.getElementById('mainModal').style.display = "block";
        refreshModal();
    }

    function closeModal() {
        document.getElementById('mainModal').style.display = "none";
    }

    // 6. 모달 내부 콘텐츠 렌더링
    function refreshModal() {
        const container = document.getElementById('sectionsContainer');
        container.innerHTML = "";

        if (currentModalType === "daily") {
            // 날짜별 카테고리 렌더링
            if (!dailyData[currentGlobalCategory]) {
                dailyData[currentGlobalCategory] = { schedule: [], class: [] };
            }
            dailyCategories.forEach(cat => {
                container.appendChild(createSectionHTML(cat.id, cat.name, dailyData[currentGlobalCategory][cat.id]));
            });
        } else {
            // 글로벌 카테고리 렌더링 (하단 메뉴)
            container.appendChild(createSectionHTML(currentGlobalCategory, "", globalData[currentGlobalCategory]));
        }
    }

    // 구역별 HTML 생성 함수
    function createSectionHTML(id, title, itemsArr) {
        const sectionDiv = document.createElement('div');
        sectionDiv.className = 'section';
        
        if (title) {
            const h3 = document.createElement('h3');
            h3.innerText = title;
            sectionDiv.appendChild(h3);
        }

        // 반장 전용 추가 폼
        const adminDiv = document.createElement('div');
        adminDiv.className = 'admin-only';
        adminDiv.style.display = isPresident ? 'block' : 'none';
        adminDiv.innerHTML = `
            <input type="text" id="text_${id}" placeholder="내용을 입력하세요...">
            <input type="file" id="file_${id}" accept="image/*" multiple>
            <button class="add-btn" onclick="addItem('${id}')">등록하기</button>
        `;
        sectionDiv.appendChild(adminDiv);

        // 등록된 데이터 리스트
        const dataList = document.createElement('div');
        const items = itemsArr || [];
        
        items.forEach((item, index) => {
            const itemCard = document.createElement('div');
            itemCard.className = 'item-card';
            
            let html = `<p style="margin:0 0 10px 0; white-space: pre-wrap;">${item.text}</p>`;
            if (item.images && item.images.length > 0) {
                html += `<div class="img-preview">`;
                item.images.forEach(imgSrc => {
                    html += `<img src="${imgSrc}" alt="첨부사진">`;
                });
                html += `</div>`;
            }
            itemCard.innerHTML = html;

            const delBtn = document.createElement('button');
            delBtn.className = 'delete-btn';
            delBtn.innerText = '삭제';
            delBtn.style.display = isPresident ? 'inline-block' : 'none';
            delBtn.onclick = () => deleteItem(id, index);
            itemCard.appendChild(delBtn);

            dataList.appendChild(itemCard);
        });

        if (items.length === 0) {
            dataList.innerHTML = `<p style="color:#888; font-size:14px; text-align:center; padding: 20px 0;">아직 등록된 내용이 없습니다.</p>`;
        }

        sectionDiv.appendChild(dataList);
        return sectionDiv;
    }

    // 7. 데이터 추가 및 삭제 로직
    async function addItem(id) {
        const textInput = document.getElementById(`text_${id}`).value;
        const fileInput = document.getElementById(`file_${id}`);
        
        if (!textInput && fileInput.files.length === 0) {
            alert("내용이나 사진을 추가해주세요.");
            return;
        }

        let imagesBase64 = [];
        if (fileInput.files.length > 0) {
            for (let file of fileInput.files) {
                const reader = new FileReader();
                const base64Promise = new Promise(resolve => {
                    reader.onload = (e) => resolve(e.target.result);
                });
                reader.readAsDataURL(file);
                imagesBase64.push(await base64Promise);
            }
        }

        const newItem = { text: textInput, images: imagesBase64 };

        if (currentModalType === "daily") {
            dailyData[currentGlobalCategory][id].push(newItem);
            localStorage.setItem('alarmiDaily', JSON.stringify(dailyData));
        } else {
            globalData[id].push(newItem);
            localStorage.setItem('alarmiGlobal', JSON.stringify(globalData));
        }

        refreshModal();
    }

    function deleteItem(id, index) {
        if (confirm("정말 이 기록을 삭제하시겠습니까?")) {
            if (currentModalType === "daily") {
                dailyData[currentGlobalCategory][id].splice(index, 1);
                localStorage.setItem('alarmiDaily', JSON.stringify(dailyData));
            } else {
                globalData[id].splice(index, 1);
                localStorage.setItem('alarmiGlobal', JSON.stringify(globalData));
            }
            refreshModal();
        }
    }

    // 초기 실행
    showTodayDate();
    renderCalendar();

    window.onclick = function(event) {
        if (event.target == document.getElementById('mainModal')) {
            closeModal();
        }
    }
</script>

</body>
</html>
