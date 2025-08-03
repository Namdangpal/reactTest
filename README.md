import React, { useState, useRef, useEffect } from "react";
import { AgGridReact } from "ag-grid-react";
import "ag-grid-community/styles/ag-grid.css";
import "ag-grid-community/styles/ag-theme-alpine.css";
import { ModuleRegistry, AllCommunityModule } from 'ag-grid-community';



ModuleRegistry.registerModules([ AllCommunityModule ]);

const initialRowData = [
  { id: 1, name: "알리", age: 20, hobby: "낚시", orderNum: 1, isActive: true, isOriginal: true, isModified: false },
  { id: 2, name: "바둑", age: 25, hobby: "게임", orderNum: 2, isActive: true, isOriginal: true, isModified: false },
  { id: 3, name: "차차", age: 30, hobby: "독서", orderNum: 3, isActive: true, isOriginal: true, isModified: false },
  { id: 4, name: "다다", age: 22, hobby: "운동", orderNum: 4, isActive: true, isOriginal: true, isModified: false },
  { id: 5, name: "라라", age: 28, hobby: "음악", orderNum: 5, isActive: true, isOriginal: true, isModified: false },
  { id: 6, name: "마마", age: 35, hobby: "요리", orderNum: 6, isActive: true, isOriginal: true, isModified: false },
  { id: 7, name: "바바", age: 27, hobby: "여행", orderNum: 7, isActive: true, isOriginal: true, isModified: false },
  { id: 8, name: "사사", age: 31, hobby: "영화", orderNum: 8, isActive: true, isOriginal: true, isModified: false },
  { id: 9, name: "아아", age: 24, hobby: "그림", orderNum: 9, isActive: true, isOriginal: true, isModified: false },
  { id: 10, name: "자자", age: 29, hobby: "프로그래밍", orderNum: 10, isActive: true, isOriginal: true, isModified: false },
];

const columnDefs = [
  {
    headerName: "선택",
    field: "checkbox",
    checkboxSelection: true,
    headerCheckboxSelection: true,
    width: 60,
    // 원본 데이터는 체크박스 비활성화
    checkboxSelection: (params) => !params.data.isOriginal,
  },
  { 
    headerName: "이름", 
    field: "name", 
    editable: true,
    width: 120
  },
  { 
    headerName: "나이", 
    field: "age", 
    editable: true, 
    type: "numericColumn",
    width: 80
  },
  { 
    headerName: "취미", 
    field: "hobby", 
    editable: true,
    width: 120
  },
  { 
    headerName: "사용여부", 
    field: "isActive", 
    editable: true,
    cellEditor: 'agSelectCellEditor',
    cellEditorParams: {
      values: ['사용', '미사용']
    },
    valueFormatter: (params) => params.value === '사용' || params.value === true ? "사용" : "미사용",
    valueParser: (params) => params.newValue === '사용' ? true : false,
    width: 100
  },
  { 
    headerName: "나열순서", 
    field: "orderNum", 
    editable: true, 
    type: "numericColumn",
    width: 100
  },
];

const App = () => {
  const [rowData, setRowData] = useState([]);
  const [originalData, setOriginalData] = useState([]);
  const gridRef = useRef();

  useEffect(() => {
    // 원본 데이터를 깊은 복사하여 저장
    const deepCopyData = JSON.parse(JSON.stringify(initialRowData));
    setOriginalData(deepCopyData);
    // rowData는 별도의 깊은 복사로 설정
    setRowData(JSON.parse(JSON.stringify(initialRowData)));
  }, []);

  // 원본 데이터와 비교하여 수정 여부 확인
  const checkModifications = () => {
    const modifiedData = rowData.filter(row => {
      if (!row.isOriginal) return false; // 추가된 데이터는 제외
      
      const originalRow = originalData.find(original => original.id === row.id);
      if (!originalRow) return false;
      
      // 각 필드별로 비교
      const nameChanged = row.name !== originalRow.name;
      const ageChanged = row.age !== originalRow.age;
      const hobbyChanged = row.hobby !== originalRow.hobby;
      const orderNumChanged = row.orderNum !== originalRow.orderNum;
      const isActiveChanged = row.isActive !== originalRow.isActive;
      
      // 하나라도 변경되었으면 수정된 것으로 간주
      const isModified = nameChanged || ageChanged || hobbyChanged || orderNumChanged || isActiveChanged;
      
      if (isModified) {
        console.log(`ID ${row.id} 수정 감지:`, {
          name: { original: originalRow.name, current: row.name, changed: nameChanged },
          age: { original: originalRow.age, current: row.age, changed: ageChanged },
          hobby: { original: originalRow.hobby, current: row.hobby, changed: hobbyChanged },
          orderNum: { original: originalRow.orderNum, current: row.orderNum, changed: orderNumChanged },
          isActive: { original: originalRow.isActive, current: row.isActive, changed: isActiveChanged }
        });
      }
      
      return isModified;
    });
    
    console.log("수정된 원본 데이터:", modifiedData);
    console.log("원본 데이터:", originalData);
    console.log("현재 데이터:", rowData);
    
    return modifiedData;
  };

  // 새 row 추가
  const addRow = () => {
    // rowData가 비어있을 때도 유일한 ID 생성
    const maxId = rowData.length > 0 ? Math.max(...rowData.map(row => row.id)) : 0;
    const newId = maxId + 1;
    
    // 원본 데이터의 최대 orderNum 찾기
    const originalRows = rowData.filter(row => row.isOriginal);
    const maxOriginalOrderNum = originalRows.length > 0 ? Math.max(...originalRows.map(row => row.orderNum)) : 0;
    
    // 새로 추가된 항목들의 개수
    const addedRows = rowData.filter(row => !row.isOriginal);
    const newOrderNum = maxOriginalOrderNum + addedRows.length + 1;
    
    // 중복 체크
    const isDuplicate = rowData.some(row => row.orderNum === newOrderNum);
    if (isDuplicate) {
      console.warn(`orderNum ${newOrderNum}가 이미 존재합니다. 다음 번호로 설정합니다.`);
      // 다음 사용 가능한 번호 찾기
      const usedOrderNums = rowData.map(row => row.orderNum).sort((a, b) => a - b);
      let nextOrderNum = maxOriginalOrderNum + 1;
      while (usedOrderNums.includes(nextOrderNum)) {
        nextOrderNum++;
      }
      newOrderNum = nextOrderNum;
    }
    
    const newRow = {
      id: newId,
      name: `새사람${newId}`,
      age: Math.floor(Math.random() * 50) + 20,
      hobby: "새취미",
      orderNum: newOrderNum,
      isActive: true,
      isOriginal: false, // 새로 추가되는 row는 isOriginal: false
      isModified: false
    };
    
    setRowData([...rowData, newRow]);
    
    // 새로 추가된 행을 편집 모드로 만들기
    setTimeout(() => {
      if (gridRef.current && gridRef.current.api) {
        const lastRowIndex = rowData.length; // 새로 추가된 행의 인덱스
        gridRef.current.api.startEditingCell({
          rowIndex: lastRowIndex,
          colKey: 'orderNum' // 나열순서 컬럼부터 편집 시작
        });
      }
    }, 100); // 약간의 지연을 두어 DOM이 업데이트된 후 편집 모드 시작
  };

  // 선택된 row 삭제 (추가된 row만 삭제 가능)
  const deleteSelectedRows = () => {
    const selectedNodes = gridRef.current.api.getSelectedNodes();
    const selectedIds = selectedNodes.map(node => node.data.id);
    
    // isOriginal이 false인 row만 삭제 (추가된 row만 삭제)
    const newRowData = rowData.filter(row => !selectedIds.includes(row.id) || row.isOriginal);
    
    // 새로 추가된 항목들의 orderNum 순서 재정렬
    const reorderedData = reorderAddedRows(newRowData);
    setRowData(reorderedData);
    
    console.log("선택된 추가 row 삭제 완료 및 orderNum 재정렬");
  };

  // 모든 추가된 row 삭제 (원본 데이터는 유지)
  const deleteAllAddedRows = () => {
    const originalData = rowData.filter(row => row.isOriginal);
    setRowData(originalData);
    console.log("모든 추가된 항목 삭제 완료. 원본 데이터만 유지됨.");
  };

  // 모든 row 선택
  const selectAllRows = () => {
    gridRef.current.api.selectAll();
  };

  // 선택 해제
  const deselectAllRows = () => {
    gridRef.current.api.deselectAll();
  };

  // 원본 데이터 복원
  const restoreOriginalData = () => {
    // originalData에서 깊은 복사하여 복원
    const deepCopyData = JSON.parse(JSON.stringify(originalData));
    setRowData(deepCopyData);
  };

  // 현재 데이터를 콘솔에 출력
  const logCurrentData = () => {
    console.log("현재 rowData:", rowData);
    console.log("원본 originalData:", originalData);
  };

  // 수정된 데이터 확인
  const checkModifiedData = () => {
    const modifiedData = checkModifications();
    alert(`수정된 원본 데이터: ${modifiedData.length}개\n추가된 데이터: ${rowData.filter(row => !row.isOriginal).length}개`);
  };

  // 새로 추가된 항목들의 orderNum 순서 재정렬
  const reorderAddedRows = (data) => {
    // 원본 데이터와 새로 추가된 데이터 분리
    const originalRows = data.filter(row => row.isOriginal);
    const addedRows = data.filter(row => !row.isOriginal);
    
    // 원본 데이터의 최대 orderNum 찾기
    const maxOriginalOrderNum = originalRows.length > 0 ? Math.max(...originalRows.map(row => row.orderNum)) : 0;
    
    // 사용 중인 orderNum 목록 생성
    const usedOrderNums = new Set(originalRows.map(row => row.orderNum));
    
    // 새로 추가된 항목들의 orderNum을 중복 없이 순차적으로 재정렬
    const reorderedAddedRows = addedRows.map((row, index) => {
      let newOrderNum = maxOriginalOrderNum + index + 1;
      
      // 중복이 있으면 다음 사용 가능한 번호 찾기
      while (usedOrderNums.has(newOrderNum)) {
        newOrderNum++;
      }
      
      usedOrderNums.add(newOrderNum);
      
      return {
        ...row,
        orderNum: newOrderNum
      };
    });
    
    // 원본 데이터와 재정렬된 추가 데이터 합치기
    const reorderedData = [...originalRows, ...reorderedAddedRows];
    
    console.log("새로 추가된 항목 orderNum 재정렬:", {
      원본_최대_orderNum: maxOriginalOrderNum,
      원본_항목_수: originalRows.length,
      추가_항목_수: addedRows.length,
      재정렬_전: addedRows.map(r => ({ id: r.id, orderNum: r.orderNum })),
      재정렬_후: reorderedAddedRows.map(r => ({ id: r.id, orderNum: r.orderNum }))
    });
    
    return reorderedData;
  };

  // 원본 데이터 무결성 확인
  const checkOriginalDataIntegrity = () => {
    console.log("=== 원본 데이터 무결성 확인 ===");
    console.log("initialRowData (상수):", initialRowData);
    console.log("originalData (상태):", originalData);
    console.log("rowData (현재):", rowData);
    
    // 원본 데이터가 변경되었는지 확인
    const isOriginalDataChanged = JSON.stringify(initialRowData) !== JSON.stringify(originalData);
    console.log("원본 데이터 변경 여부:", isOriginalDataChanged);
    
    if (isOriginalDataChanged) {
      console.warn("⚠️ 원본 데이터가 변경되었습니다!");
    } else {
      console.log("✅ 원본 데이터가 보호되고 있습니다.");
    }
  };

  // 등록 버튼 클릭 시 데이터 전송
  const handleSubmit = () => {
    const modifiedData = checkModifications();
    const addedData = rowData.filter(row => !row.isOriginal);
    
    const submitData = {
      modified: modifiedData.map(row => ({
        id: row.id,
        name: row.name,
        age: row.age,
        hobby: row.hobby,
        orderNum: row.orderNum,
        isActive: row.isActive
      })),
      added: addedData.map(row => ({
        name: row.name,
        age: row.age,
        hobby: row.hobby,
        orderNum: row.orderNum,
        isActive: row.isActive
      })),
      total: [...modifiedData, ...addedData]
    };
    
    console.log("전송할 데이터:", submitData);
    
    // 백엔드로 POST 요청 전송
    fetch('/api/submit-data', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(submitData)
    })
    .then(response => response.json())
    .then(data => {
      console.log('서버 응답:', data);
      alert('데이터가 성공적으로 전송되었습니다!');
    })
    .catch(error => {
      console.error('전송 오류:', error);
      alert('데이터 전송 중 오류가 발생했습니다.');
    });
  };

  // 셀 편집 완료 시 수정 여부 업데이트
  const onCellValueChanged = (params) => {
    const { data, field, newValue, oldValue } = params;
    
    console.log("셀 값 변경 감지:", { field, newValue, oldValue, dataId: data.id });
    
    if (data.isOriginal) {
      const originalRow = originalData.find(original => original.id === data.id);
      if (originalRow) {
        // 각 필드별로 비교
        const nameChanged = data.name !== originalRow.name;
        const ageChanged = data.age !== originalRow.age;
        const hobbyChanged = data.hobby !== originalRow.hobby;
        const orderNumChanged = data.orderNum !== originalRow.orderNum;
        const isActiveChanged = data.isActive !== originalRow.isActive;
        
        // 하나라도 수정되면 수정된 것으로 표시
        const isModified = nameChanged || ageChanged || hobbyChanged || orderNumChanged || isActiveChanged;
        
        // 수정 여부 업데이트
        const updatedRowData = rowData.map(row => 
          row.id === data.id ? { ...row, isModified } : row
        );
        setRowData(updatedRowData);
        
        // 수정된 경우 콘솔에 로그 출력
        if (isModified) {
          console.log(`ID ${data.id}의 데이터가 수정되었습니다.`);
          console.log('원본:', originalRow);
          console.log('수정된 데이터:', data);
          console.log('변경된 필드들:', {
            name: nameChanged ? `${originalRow.name} → ${data.name}` : '변경없음',
            age: ageChanged ? `${originalRow.age} → ${data.age}` : '변경없음',
            hobby: hobbyChanged ? `${originalRow.hobby} → ${data.hobby}` : '변경없음',
            orderNum: orderNumChanged ? `${originalRow.orderNum} → ${data.orderNum}` : '변경없음',
            isActive: isActiveChanged ? `${originalRow.isActive} → ${data.isActive}` : '변경없음'
          });
        }
      }
    }
  };

  return (
    <div style={{ padding: "20px" }}>
      <div style={{ marginBottom: "20px" }}>
        <button 
          onClick={addRow}
          style={{ 
            marginRight: "10px", 
            padding: "8px 16px", 
            backgroundColor: "#4CAF50", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          행 추가
        </button>
        <button 
          onClick={deleteSelectedRows}
          style={{ 
            marginRight: "10px", 
            padding: "8px 16px", 
            backgroundColor: "#f44336", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          선택된 행 삭제
        </button>
        <button 
          onClick={deleteAllAddedRows}
          style={{ 
            marginRight: "10px", 
            padding: "8px 16px", 
            backgroundColor: "#ff9800", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          추가된 행만 삭제
        </button>
        <button 
          onClick={selectAllRows}
          style={{ 
            marginRight: "10px", 
            padding: "8px 16px", 
            backgroundColor: "#2196F3", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          전체 선택
        </button>
        <button 
          onClick={deselectAllRows}
          style={{ 
            marginRight: "10px", 
            padding: "8px 16px", 
            backgroundColor: "#9E9E9E", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          선택 해제
        </button>
        <button 
          onClick={restoreOriginalData}
          style={{ 
            marginRight: "10px", 
            padding: "8px 16px", 
            backgroundColor: "#673AB7", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          원본 데이터 복원
        </button>
        <button 
          onClick={logCurrentData}
          style={{ 
            marginRight: "10px", 
            padding: "8px 16px", 
            backgroundColor: "#607D8B", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          데이터 확인 (콘솔)
        </button>
        <button 
          onClick={checkModifiedData}
          style={{ 
            marginRight: "10px",
            padding: "8px 16px", 
            backgroundColor: "#E91E63", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          수정/추가 확인
        </button>
        <button 
          onClick={checkOriginalDataIntegrity}
          style={{ 
            marginRight: "10px",
            padding: "8px 16px", 
            backgroundColor: "#795548", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          원본데이터 확인
        </button>
        <button 
          onClick={() => {
            const reorderedData = reorderAddedRows(rowData);
            setRowData(reorderedData);
            console.log("수동으로 orderNum 재정렬 완료");
          }}
          style={{ 
            marginRight: "10px",
            padding: "8px 16px", 
            backgroundColor: "#009688", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          orderNum 재정렬
        </button>
        <button 
          onClick={() => {
            const orderNums = rowData.map(row => row.orderNum);
            const duplicates = orderNums.filter((num, index) => orderNums.indexOf(num) !== index);
            if (duplicates.length > 0) {
              const duplicateDetails = duplicates.map(num => {
                const duplicateRows = rowData.filter(row => row.orderNum === num);
                const names = duplicateRows.map(row => row.name).join(', ');
                return `orderNum ${num}: ${names}`;
              }).join('\n');
              alert(`중복된 orderNum 발견:\n${duplicateDetails}`);
            } else {
              alert("중복된 orderNum이 없습니다.");
            }
            console.log("현재 orderNum 목록:", orderNums);
          }}
          style={{ 
            marginRight: "10px",
            padding: "8px 16px", 
            backgroundColor: "#FF9800", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer"
          }}
        >
          orderNum 중복 체크
        </button>
        <button 
          onClick={handleSubmit}
          style={{ 
            padding: "8px 16px", 
            backgroundColor: "#FF5722", 
            color: "white", 
            border: "none", 
            borderRadius: "4px",
            cursor: "pointer",
            fontSize: "16px",
            fontWeight: "bold"
          }}
        >
          등록
        </button>
      </div>
      
      <div className="ag-theme-alpine" style={{ height: 400, width: "100%" }}>
        <AgGridReact
          ref={gridRef}
          rowData={rowData}
          columnDefs={columnDefs}
          rowSelection="multiple"
          suppressRowClickSelection={true}
          domLayout="autoHeight"
          theme="legacy"
          onCellValueChanged={onCellValueChanged}
          onCellEditingStopped={(params) => {
            console.log("셀 편집 중지:", params);
            // orderNum 편집 완료 시 중복 체크
            if (params.column.colId === 'orderNum') {
              const { data, newValue, oldValue } = params;
              console.log("orderNum 편집 완료:", { newValue, oldValue, dataId: data.id });
              
              // 중복 체크
              const duplicateRow = rowData.find(row => 
                row.id !== data.id && row.orderNum === newValue
              );
              
              if (duplicateRow) {
                console.log("중복 발견:", duplicateRow);
                alert(`경고: 나열순서 ${newValue}는 이미 "${duplicateRow.name}" 항목에서 사용 중입니다.\n원래 값 ${oldValue}로 되돌립니다.`);
                
                // 원래 값으로 되돌리기
                setRowData(prevData => 
                  prevData.map(row => 
                    row.id === data.id ? { ...row, orderNum: oldValue } : row
                  )
                );
              } else {
                console.log("중복 없음, 변경 허용");
              }
            }
          }}
        />
      </div>
      
      <div style={{ marginTop: "20px", fontSize: "14px", color: "#666" }}>
        총 {rowData.length}개의 행이 있습니다.
        <br />
        <strong>사용법:</strong>
        <ul>
          <li>모든 컬럼은 더블클릭하여 편집할 수 있습니다.</li>
          <li>나열순서는 숫자로 입력하세요.</li>
          <li>사용여부는 드롭다운에서 선택하세요.</li>
          <li>원본 데이터는 수정만 가능하고, 추가된 행만 삭제할 수 있습니다.</li>
          <li>등록 버튼을 클릭하면 수정된 데이터와 추가된 데이터를 구별하여 서버로 전송합니다.</li>
        </ul>
      </div>
    </div>
  );
};

export default App;
