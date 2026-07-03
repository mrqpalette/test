// 상단 헤더(rec_top)를 제외하고 rec_로 시작하는 모든 데이터 행(tr) 선택
const rows = document.querySelectorAll('tr[id^="grid_user_list_rec_"]:not(#grid_user_list_rec_top)');

const result = Array.from(rows).map(row => {
    // 각 행에서 2번째(사번), 3번째(이름), 6번째(부서) 열(td) 선택
    const empIdTd = row.querySelector('td[col="2"] div');
    const nameTd = row.querySelector('td[col="3"] div');
    const deptTd = row.querySelector('td[col="6"] div');

    return {
        '사번': empIdTd ? empIdTd.textContent.trim() : '',
        '이름': nameTd ? nameTd.textContent.trim() : '',
        '부서명': deptTd ? deptTd.textContent.trim() : ''
    };
});

// 결과를 깔끔한 테이블 형태로 콘솔에 출력
console.table(result);
