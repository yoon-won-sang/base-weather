ECharts에서 바 차트와 다른 차트(선, 스캐터)를 서로 다른 축에 배치하려면 **xAxisIndex**와 **yAxisIndex**를 활용하여 데이터 시리즈와 축을 각각 매핑해야 합니다.

바 차트가 전체 시간 축의 영향을 받지 않게 하거나, 반대로 바 차트만 고유한 축을 갖게 하려면 다음과 같이 설정합니다.

1. xAxis를 2개 정의

X축을 2개 선언하고, 바 차트용 X축을 별도로 분리합니다.

option = {
  xAxis: [
    {
      type: 'time', // 선/스캐터 차트용 (기본)
      id: 'mainAxis',
      // ... 범위 설정
    },
    {
      type: 'time', // 바 차트용 (별도)
      id: 'barAxis',
      show: false,  // 필요에 따라 숨김
      // ... 범위 설정
    }
  ],
  series: [
    {
      type: 'line',
      xAxisIndex: 0, // 첫 번째 축 사용
      data: [...]
    },
    {
      type: 'bar',
      xAxisIndex: 1, // 두 번째 축 사용
      data: [...]
    }
  ]
};

2. 핵심: '축 동기화(link)' 또는 '수동 매핑'

축을 2개로 나누면, 바 차트를 끄고 켤 때 바 차트용 X축(xAxisIndex: 1)은 데이터가 사라져도 영향을 받 하지만 두 축이 서로 다르게 움직이면 차트가 깨지기 때문에 두 축의 범위를 일치시켜야 합니다.

방법 A: axisPointer의 link 활용 (가장 추천)

두 축을 논리적으로 묶어버리면, 한쪽 축의 범위가 바뀌어도 다른 쪽이 똑같이 따라갑니다.

axisPointer: {
  link: { xAxisIndex: 'all' } // 0번 축과 1번 축을 똑같이 움직이게 함
}

방법 B: legendselectchanged 이벤트로 축 범위 강제 동기화

바 차트를 숨길 때 발생하는 축 범위 변화를 완전히 차단하고 싶다면, 이벤트를 이용해 축의 min, max를

myChart.on('legendselectchanged', function (params) {
  // 레전드가 바뀔 때마다 실행됨
  myChart.setOption({
    xAxis: [
      { id: 'mainAxis', min: '2026-09-01', max: '2026-10-01' },
      { id: 'barAxis', min: '2026-09-01', max: '2026-10-01' }
    ]
  });
});

왜 이 방법이 도움이 되나요?

바 차트만 별도의 축(xAxisIndex: 1)을 사용하면, 바 차트 시리즈의 데이터 변화가 선 차트(0번 축)의 범위 계산 로직에 직접적인 영향을 주지 않게 됩니다.

- 선 차트: 0번 축만 바라봄
- 바 차트: 1번 축만 바라봄
- 결합: axisPointer.link를 통해 두 축을 동기화하여 시각적으로는 하나의 축처럼 보이게 유지

주의사항

1. 데이터 정렬: 두 축의 min, max 값이 완전히 일치해야 합니다. 그렇지 않으면 선 차트의 점과 바 차트에 놓이게 됩니다.
2. show: false 활용: 만약 바 차트용 축이 보이지 않길 원하시면 두 번째 xAxis에 show: false를 추가하

이렇게 구성하면 레전드를 숨길 때 바 차트 시리즈는 xAxisIndex: 1에서 사라지지만, xAxisIndex: 0(선   있으므로 차트 전체가 옆으로 덜컥거리는 현상을 확실하게 방지할 수 있습니다.
