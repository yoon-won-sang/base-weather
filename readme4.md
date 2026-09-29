toolbox.feature.dataZoom.yAxisIndex: [0]으로도 해결되지 않는다면, 이는 ECharts 브러쉬 줌(dataZoom)이 축의 '범위 변화'를 해석하는 방식 때문입니다.

브러쉬 줌은 실행 시 해당 축의 모든 데이터 범위를 재탐색합니다. 이때 고정된 Y축(보조축)이 "나는 범위가 고정되어 있어!"라고 선언하고 있어도, 브러쉬 줌이 사각형 영역을 잡는 순간 **"내가 잡은 영역(Y좌표)에 맞춰 이 축의 범위도 바꿔야지"**라는 줌의 기본 로직이 고정값(min/max)보다 우선순위를 갖게 되어 충돌하는 것입니다.

이 문제를 우회하여 **보조축을 '철벽 고정'**하는 확실한 방법 2가지를 제안합니다.

1. dataZoom의 filterMode 변경 (가장 효과적)

ECharts의 dataZoom은 기본적으로 데이터를 필터링(filterMode: 'filter')합니다. 즉, 줌을 하면 차트 내부의 데이터를 잘라내는데, 이때 고정된 Y축이 줌 영역 밖의 데이터를 참조하려다 충돌이 납니다.

이를 weakFilter나 empty로 바꾸면 축이 데이터를 잘라내지 않고 시각적인 범위만 조정하므로 고정값 충돌이 줄어듭니다.

dataZoom: [
  {
    type: 'inside',
    xAxisIndex: [0, 1],
    yAxisIndex: [0],
    filterMode: 'empty' // 'filter' 대신 'empty'를 사용해보세요
  }
]

2. toolbox에서 브러쉬 줌 대신 setOption으로 제어

툴박스의 기본 dataZoom 기능이 고정축과 계속 싸운다면, 툴박스의 기능을 아예 끄고 dataZoom 이벤트에서 보조축의 고정값을 재선언하는 것이 가장 강력합니다.

myChart.on('dataZoom', function (params) {
  // 줌이 일어난 직후 무조건 보조축을 고정값으로 덮어씌움
  myChart.setOption({
    yAxis: [
      { /* 주축 설정 */ },
      {
        id: 'barAxis', // 보조축 ID
        min: 0,
        max: 100,
        // 강제로 고정값을 다시 입힘
      }
    ]
  });
});

3. 혹시 toolbox에 brush 기능도 같이 쓰시나요?

만약 toolbox.feature.brush를 사용 중이라면, brush와 dataZoom은 서로 축을 제어하려는 욕구가 매우 강합니다.

이 경우, 아예 브러쉬 기능에서 보조축을 배제해야 합니다.

toolbox: {
  feature: {
    dataZoom: {
      yAxisIndex: [0] // 보조축 인덱스는 여기에 절대 포함하지 마세요
    }
  }
},
brush: {
  xAxisIndex: [0, 1],
  yAxisIndex: [0] // 여기서도 1번(보조축)은 뺍니다.
}

그래도 안 될 때 확인해야 할 핵심 질문

1. 보조축(yAxisIndex: 1)이 아예 안 움직이는 것인가요, 아니면 줌 자체가 안 먹히는 것인가요?
2. 보조축 설정에 scale: true가 들어가 있지는 않은가요? (scale: true는 고정값을 무시하고 데이터를 따라가려는 속성이 강합니다. 반드시 false여야 합니다.)
3. min: 0, max: 100을 설정할 때 혹시 문자열로 되어 있나요? (min: '0', max: '100' 보다는 숫자 0, 100으로 명확히 입력되어 있는지 확인해 주세요.)
