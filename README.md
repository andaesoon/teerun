# teerun
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>나만의 걸음 기록 웹앱</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            background-color: #f3f4f6;
        }
    </style>
</head>
<body class="p-4 sm:p-8">
    <div class="max-w-4xl mx-auto bg-white p-6 rounded-xl shadow-lg">

        <header class="mb-8 text-center">
            <h1 class="text-3xl font-bold text-gray-800">나만의 걸음 기록 웹앱 🚶‍♂️</h1>
            <p class="text-gray-500 mt-2">오늘의 걸음을 기록하고 통계를 확인하세요!</p>
        </header>

        <section class="grid grid-cols-1 sm:grid-cols-3 gap-4 mb-8">
            <div class="bg-blue-100 p-4 rounded-lg text-center">
                <p class="text-sm text-blue-600">총 걸음 수</p>
                <p id="total-steps" class="text-2xl font-bold text-blue-800 mt-1">0</p>
            </div>
            <div class="bg-green-100 p-4 rounded-lg text-center">
                <p class="text-sm text-green-600">총 이동 거리 (km)</p>
                <p id="total-distance" class="text-2xl font-bold text-green-800 mt-1">0</p>
            </div>
            <div class="bg-purple-100 p-4 rounded-lg text-center">
                <p class="text-sm text-purple-600">평균 걸음 수</p>
                <p id="average-steps" class="text-2xl font-bold text-purple-800 mt-1">0</p>
            </div>
        </section>

        <section class="bg-gray-50 p-6 rounded-lg mb-8">
            <h2 class="text-xl font-semibold text-gray-700 mb-4">오늘의 기록 추가하기</h2>
            <form id="record-form" class="space-y-4">
                <div>
                    <label for="steps" class="block text-sm font-medium text-gray-700">걸음 수 (걸음)</label>
                    <input type="number" id="steps" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 p-2">
                </div>
                <div>
                    <label for="distance" class="block text-sm font-medium text-gray-700">거리 (km)</label>
                    <input type="number" id="distance" step="0.1" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 p-2">
                </div>
                <div>
                    <label for="date" class="block text-sm font-medium text-gray-700">날짜</label>
                    <input type="date" id="date" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 p-2">
                </div>
                <button type="submit" class="w-full bg-indigo-600 text-white font-bold py-2 px-4 rounded-md hover:bg-indigo-700 transition-colors">기록 추가</button>
            </form>
        </section>

        <section>
            <h2 class="text-xl font-semibold text-gray-700 mb-4">내 기록</h2>
            <div class="overflow-x-auto">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-50">
                        <tr>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">날짜</th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">걸음 수</th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">거리 (km)</th>
                        </tr>
                    </thead>
                    <tbody id="record-table-body" class="bg-white divide-y divide-gray-200">
                        </tbody>
                </table>
            </div>
        </section>
    </div>

    <script>
        // DOM 요소 가져오기
        const recordForm = document.getElementById('record-form');
        const recordTableBody = document.getElementById('record-table-body');
        const totalStepsEl = document.getElementById('total-steps');
        const totalDistanceEl = document.getElementById('total-distance');
        const averageStepsEl = document.getElementById('average-steps');
        const dateInput = document.getElementById('date');

        // 오늘 날짜로 기본값 설정
        dateInput.value = new Date().toISOString().substring(0, 10);

        // 데이터 저장 및 불러오기 함수
        const getRecords = () => {
            const records = localStorage.getItem('walkingRecords');
            return records ? JSON.parse(records) : [];
        };

        const saveRecords = (records) => {
            localStorage.setItem('walkingRecords', JSON.stringify(records));
        };

        // 통계 계산 및 화면 표시 함수
        const updateStats = (records) => {
            const totalSteps = records.reduce((sum, record) => sum + record.steps, 0);
            const totalDistance = records.reduce((sum, record) => sum + record.distance, 0);
            const averageSteps = records.length > 0 ? Math.round(totalSteps / records.length) : 0;

            totalStepsEl.textContent = totalSteps.toLocaleString();
            totalDistanceEl.textContent = totalDistance.toFixed(2);
            averageStepsEl.textContent = averageSteps.toLocaleString();
        };

        // 기록 테이블 렌더링 함수
        const renderRecords = (records) => {
            recordTableBody.innerHTML = ''; // 테이블 초기화
            records.sort((a, b) => new Date(b.date) - new Date(a.date)); // 최신 기록이 위로 오도록 정렬

            records.forEach(record => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${record.date}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${record.steps.toLocaleString()}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${record.distance.toFixed(2)}</td>
                `;
                recordTableBody.appendChild(row);
            });
        };

        // 앱 초기화 및 데이터 로드
        const initApp = () => {
            const records = getRecords();
            updateStats(records);
            renderRecords(records);
        };

        // 폼 제출 이벤트 핸들러
        recordForm.addEventListener('submit', (e) => {
            e.preventDefault(); // 기본 폼 제출 동작 방지

            const newRecord = {
                steps: parseInt(document.getElementById('steps').value),
                distance: parseFloat(document.getElementById('distance').value),
                date: document.getElementById('date').value,
            };

            const records = getRecords();
            records.push(newRecord);
            saveRecords(records);

            // UI 업데이트
            updateStats(records);
            renderRecords(records);

            // 폼 필드 초기화
            recordForm.reset();
            dateInput.value = new Date().toISOString().substring(0, 10);
        });

        // 페이지 로드 시 앱 초기화
        window.onload = initApp;
    </script>
</body>
</html>
