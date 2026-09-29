2. 해결을 위한 axisPointer 설정 전략

A. axisPointer를 고정 모드로 변경 (추천)

axisPointer가 데이터 변화에 민감하게 반응하지 않도록 설정을 바꿔보세요.

tooltip: {
  trigger: 'axis',
  axisPointer: {
    type: 'cross', // 또는 'line'
    snap: true,    // false로 설정하면 마우스 위치를 데이터에 강제로 맞추지 않음
    animation: false, // 애니메이션을 끄면 축 재계산 딜레이가 줄어듦
  }
}

B. axisPointer의 간섭을 차단

차트 설정(option)에서 axisPointer가 축을 직접 제어하지 못하게 막을 수 있습니다.

xAxis: {
  type: 'time',
  axisPointer: {
    show: true,
    // 이 옵션이 핵심입니다.
    // tooltip이나 다른 컴포넌트가 축을 건드리지 못하게 합니다.
    link: { xAxisIndex: 'all' }
  }
}

C. tooltip 설정 분리

만약 tooltip 때문에 축이 요동친다면, tooltip 설정을 아래와 같이 수정하여 axisPointer가 축의 범위를 계산할 때 '모든 데이터'를 보도록 강제할 수 있습니다.

tooltip: {
  trigger: 'axis',
  // 축 포인터가 데이터가 사라져도 범위를 유지하도록 힌트를 줌
  axisPointer: {
    animation: false,
    label: {
      show: true,
      // 포맷터를 고정하면 계산 로직이 단순해짐
      formatter: function (params) {
        return params.value;
      }
    }
  }
}

3. 요약: 왜 axisPointer가 범인인가?

- **trigger: 'axis'**는 사용자에게는 편리한 툴팁 기능을 제공하지만, ECharts 엔진 내부적으로는 **"  즈의 시간 범위를 기준으로 X축의 스케일을 구성하라"**는 명령을 계속 내리는 것과 같습니다.
- 레전드를 클릭해 데이터를 빼면, 그 즉시 axisPointer는 "데이터가 줄었으니 X축의 Max도 줄여도 되겠

실제 적용해보실 때 체크리스트:
1. tooltip의 trigger를 'axis'에서 'item'으로 바꿔보세요. (만약 툴팁이 데이터 하나하나에만 떠도 된 )
2. axisPointer의 snap 옵션을 false로 바꿔보세요.
3. 그래도 안 된다면 option.axisPointer 설정 전체를 차트에서 제거하고 툴팁만 남겨보세요.

이렇게 설정하면 axisPointer가 축의 범위를 강제로 재계산하는 빈도를 확실히 줄일 수 있습니다. 적용해요!
