<script>
  import SuperOdometer from "./SuperOdometer.svelte";
  import { getContext } from "svelte"

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  export let title
  export let gaugeMinSize
  export let min
  export let max
  export let value = 200
  export let arcSize

  export let trackColor
  export let valueColor
  export let needleColor
  export let centerColor
  export let tickLabelColor
  export let needleAnimation

  // Properties so set value Label options
  export let valueSize

  // Properties to set Major and Minor Ticks
  export let showTicks
  export let showTickLabels
  export let majorTicks 
  export let minorTicks

  // Rule out invalid values 
  $: max = (max > min) ? max : min + 1
  $: _value = (min <= Number(value)) && (Number(value) <= max ) ? Number(value) : 0;

  $: minorTicks = (minorTicks > 0) ? minorTicks : 1;
  $: majorTicks = (majorTicks > 0) ? majorTicks : 1;

</script>

<div use:styleable={$component.styles}>
      <SuperOdometer 
        {title}
        {min}
        {max}
        value={_value}
        {needleAnimation}
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
        {gaugeMinSize}
      />
</div>
