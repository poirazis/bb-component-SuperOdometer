<script>
	import { tweened, spring } from 'svelte/motion';
	import { backInOut } from 'svelte/easing';
	import { arc as d3arc } from 'd3-shape';
	import { scaleLinear } from 'd3-scale';
    import { writable } from 'svelte/store';

	export let title = ""
    export let arcSize = 120
	export let gaugeMinSize

	export let min = 0
	export let max = 1000
	export let value = 350
    export let needleAnimation
    export let trackColor  
    export let valueColor
    export let needleColor  
    export let centerColor  
    export let tickLabelColor  

	export let valueSize = "M"

	export let showTicks = false
  	export let showTickLabels = false
  	export let majorTicks = 10
  	export let minorTicks = 0

	// Array of configuration settigns for each Odometer variation
	let gaugeTypeMap = [];
	gaugeTypeMap[90] =  { startAngle: -90, endAngle:0, innerRadius: 232, outerRadius:236, innerArcRadius: 96, cornerRadius:10, canvas: { width:256, height: 256 }, pivot: {x: 236, y: 236}, valuePos: {x: 200, y: 208} }
	gaugeTypeMap[120] = { startAngle: -60, endAngle:60, innerRadius: 124, outerRadius:128, innerArcRadius: 48, cornerRadius:10, canvas: { width:256, height: 148 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 108} }
	gaugeTypeMap[180] = { startAngle: -90, endAngle:90, innerRadius: 124, outerRadius:128, innerArcRadius: 48, cornerRadius:10, canvas: { width:256, height: 148 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 120} }
	gaugeTypeMap[240] = { startAngle: -120, endAngle:120, innerRadius: 124, outerRadius:128, innerArcRadius: 48, cornerRadius:10, canvas: { width:256, height: 192 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 120} }
	gaugeTypeMap[270] = { startAngle: -180, endAngle:90, innerRadius: 124, outerRadius:128, innerArcRadius: 48, cornerRadius:10, canvas: { width:256, height: 276 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 120} }
	gaugeTypeMap[360] = { startAngle: 0, endAngle:360, innerRadius: 126, outerRadius:128, innerArcRadius: 48, cornerRadius:10, canvas: { width:256, height: 276 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 128} }
	
	let gaugeConfig
	$: gaugeConfig = gaugeTypeMap[arcSize];
	
	$: startAngle = gaugeConfig.startAngle;
	$: endAngle = gaugeConfig.endAngle;
	$: outerRadius = gaugeConfig.outerRadius;
	$: innerRadius = gaugeConfig.innerRadius;
	$: pivot = gaugeConfig.pivot;
	$: canvas = gaugeConfig.canvas;
	$: innerArcRadius = gaugeConfig.innerArcRadius;
	$: valuePos = gaugeConfig.valuePos;

	// An array with the angle and label of each tick
	let _majorTicks = []
	let _minorTicks = []

    let _value = writable(); 
    $:  _value.set(Number(value));

    $: if ( needleAnimation === "spring") { 
            _value = spring(min, { stiffness: .1 });
        } else if ( needleAnimation === "tweened" ) {
            _value = tweened(min, { easing: backInOut, duration: 750 });
        }

	$: scale = scaleLinear()
		.domain([min, max])
		.range([startAngle, endAngle]);
	
	$: min, max, generateTicks( majorTicks , minorTicks , gaugeConfig )
	
	$: valueAngle = scale($_value);

	$: _textBaseline = (arcSize == 360) ? "middle" : "bottom"
	
	$: trackArc = d3arc()
		.innerRadius(innerRadius)
		.outerRadius(outerRadius)
		.startAngle(startAngle * Math.PI / 180)
		.endAngle(endAngle * Math.PI / 180)
		.cornerRadius(0);

	$: innerArc = d3arc()
		.innerRadius(0)
		.outerRadius(innerArcRadius)
		.startAngle(startAngle * Math.PI / 180)
		.endAngle(endAngle * Math.PI / 180)
		.cornerRadius(2);

	function generateTicks ( majorTicks, minorTicks ) {
		let _majorStep = ( max - min ) / majorTicks;
		let _minorStep = _majorStep / minorTicks;
		let _pos = Number(min) || 0;
		let _minorPos = _pos;

		_majorTicks = [];
		_minorTicks = [];
		while ( _pos <= max ) {
			_majorTicks.push({ angle:scale(Number(_pos)) , label: Math.round(_pos) });
			while ((_minorPos <= _pos + _majorStep) && _minorPos <= max )
			{
				_minorTicks.push ( { angle: scale(_minorPos) , label: ""} )
				_minorPos += _minorStep
			}
			_pos += _majorStep;
		}

        if ( arcSize === "360" ) {
            _majorTicks.shift();
            _minorTicks.shift();
        }
	}

</script>

<div class="svg-box" 
    style:min-width={gaugeMinSize}
    style:--needleColor={needleColor || "var(--spectrum-global-color-gray-400)" }
    style:--valueColor={valueColor || "var(--spectrum-global-color-gray-700)" }
    style:--trackColor={trackColor || "var(--spectrum-global-color-gray-400)" }
    style:--centerColor={centerColor || "var(--spectrum-alias-background-color-default)" }
    style:--tickLabelColor={tickLabelColor || "var(--spectrum-global-color-gray-600)" }
    >

	<svg class="svg-box-content" viewbox="-2, -2, { canvas.width + 4 }, { canvas.height + 4} " version="1.1" xmlns="http://www.w3.org/2000/svg">
		<path d={trackArc()} class="track" transform="translate({pivot.x}, {pivot.y})" />

		<polygon 
			class="needle" 
			points="{pivot.x}, 6 {pivot.x + 2},{pivot.y - innerArcRadius - 2} {pivot.x - 2},{pivot.y - innerArcRadius - 2}" transform="rotate({valueAngle} {pivot.x}, {pivot.y})" />
        <path 
            d={innerArc()} 
            class="innerArc" 
            transform="translate({pivot.x}, {pivot.y})" 
        />

		{#if showTicks && _majorTicks.length > 0}
			{#each _minorTicks as tick}
				<line class="minor-tick" x1={pivot.x} y1={2} x2={pivot.x} y2={10} transform="rotate({Number(tick.angle)} {pivot.x} {pivot.y})"/>
			{/each}
			
			{#each _majorTicks as tick}
				<line class="tick" x1={pivot.x} y1={0} x2={pivot.x} y2={16} transform="rotate({Number(tick.angle)} {pivot.x} {pivot.y})"/>
				{#if showTickLabels}
					<text class="tick-label" x={pivot.x} y={26} transform="rotate({tick.angle} {pivot.x} {pivot.y})"> {tick.label} </text>
				{/if}
			{/each}
		{/if}

        <text class="spectrum-Heading spectrum-Heading--size{valueSize}" dominant-baseline={_textBaseline} transform="translate({valuePos.x} {valuePos.y})">
            {Math.round($_value)}
        </text>

		<text class="title" transform="translate({canvas.width / 2} {canvas.height})">
			{title.toUpperCase()}
		</text>
	</svg>
</div>


<style>
	.svg-box {
		width: 100%;
		min-width: 128px;
		position: relative;
	}

	.svg-box-content {
		position: relative;
		top: 0;
		left: 0;
	}
	.needle {
		fill: var(--needleColor);
	}

	.track {
		fill: var(--trackColor);
	}

	.innerArc {
		stroke: var(--needleColor); /* var(--spectrum-global-color-gray-400); */
		stroke-width: 1px;
		fill: var(--centerColor);
	}

	.tick {
		stroke: var(--trackColor);
		stroke-width: 2px;
		fill: none;
	}

	.minor-tick {
		stroke: var(--trackColor);
		stroke-width: 1px;
		fill: none;
	}
	
	.tick-label {
		fill: var(--tickLabelColor);
		font-weight: 400;
		font-size: 0.6rem;
		text-anchor: middle;
	}

	.spectrum-Heading {
		text-anchor: middle;
		fill: var(--valueColor);
	}

	.title {
		text-anchor: middle;
		font-size: 0.75rem;
		fill: var(--spectrum-global-color-gray-600);
	}
</style>