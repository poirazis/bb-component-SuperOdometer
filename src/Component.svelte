<script>
  import SuperOdometer from "./SuperOdometer.svelte";
  import { getContext } from "svelte";

  const { styleable } = getContext("sdk");
  const component = getContext("component");

  export let title;
  export let min = 0;
  export let max = 100;
  export let currentValue = 20;
  export let precision;
  export let arcSize;

  export let trackColor;
  export let valueColor;
  export let needleColor;
  export let centerColor;
  export let tickLabelColor;
  export let animationType;
  export let backdropColor = "var(--spectrum-global-color-gray-50)";

  export let prefix;
  export let suffix;
  // Properties so set value Label options
  export let valueSize;

  // Properties to set Major and Minor Ticks
  export let showTicks;
  export let showTickLabels;
  export let majorTicks;
  export let minorTicks;

  // Rule out invalid values
  $: max = max > min ? max : min + 1;
  $: _value =
    min <= Number(currentValue) && Number(currentValue) <= max
      ? Number(currentValue)
      : 0;

  $: minorTicks = minorTicks > 0 ? minorTicks : 1;
  $: majorTicks = majorTicks > 0 ? majorTicks : 1;
  $: $component.styles = {
    ...$component.styles,
    normal: {
      width: "400px",
      ...$component.styles.normal,
    },
  };
</script>

<div use:styleable={$component.styles}>
  <SuperOdometer
    {title}
    {min}
    {max}
    value={_value}
    {prefix}
    {suffix}
    {precision}
    {animationType}
    {valueColor}
    {trackColor}
    {needleColor}
    {centerColor}
    {tickLabelColor}
    {showTicks}
    {showTickLabels}
    {majorTicks}
    {minorTicks}
    {valueSize}
    {arcSize}
    {backdropColor}
  />
</div>
