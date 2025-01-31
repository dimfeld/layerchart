<script lang="ts">
  import { getContext } from 'svelte';
  import { forceSimulation, type Force } from 'd3-force';

  const { data } = getContext('LayerCake');

  export let forces: Record<string, Force<any, any>>;

  export let alpha = 1;
  export let alphaTarget = 0;
  export let alphaDecay = 1 - Math.pow(0.001, 1 / 300);
  export let alphaMin = 0.001;

  export let velocityDecay = 0.4;

  /** Clone data since simulation mutates original */
  export let cloneData = false;

  let _static = false;
  /** If true, will only update nodes after simulation has completed */
  export { _static as static };

  let simulation = forceSimulation(cloneData ? structuredClone($data) : $data);

  $: {
    simulation
      .alpha(alpha)
      .alphaTarget(alphaTarget)
      .alphaMin(alphaMin)
      .alphaDecay(alphaDecay)
      .velocityDecay(velocityDecay)
      .restart();

    if (_static) {
      // TODO: Not sure why it needs to be recreated when static
      simulation = forceSimulation(cloneData ? structuredClone($data) : $data);
      simulation.stop();

      Object.entries(forces).forEach(([name, force]) => {
        simulation.force(name, force);
      });

      for (
        let i = 0,
          n = Math.ceil(Math.log(simulation.alphaMin()) / Math.log(1 - simulation.alphaDecay()));
        i < n;
        ++i
      ) {
        simulation.tick();
      }

      nodes = simulation.nodes();
    } else {
      // When variables change, set forces and restart the simulation
      Object.entries(forces).forEach(([name, force]) => {
        simulation.force(name, force);
      });
    }
  }

  let nodes = [];
  simulation.on('tick', () => {
    nodes = simulation.nodes();
  });
</script>

<slot {nodes} {simulation} />
