<script>
  import { onMount } from "svelte";
  // import { LayerCake, Svg } from "layercake";
  import Icon from "../helpers/Icon.svelte";
  // import sampleData from "../../data/demo/sample.csv";
  // import copyData from "../../data/demo/copy.json";

  const points = [
    { x: 0, y: 1 },
    { x: 10, y: 5 },
    { x: 15, y: 10 },
  ];
  const r = 6;
  const padding = { top: r * 2, right: r * 2, bottom: r * 2, left: r * 2 };
</script>

<section id="demo">
  <!-- copy from google doc -->
  <div>
    <h1>Sam's Test Project</h1>
    <p>Using the Pudding's starter template for Svelte <Icon name="feather" /></p>
  </div>

  <div>
    <div>
      <p><input placeholder="input" /></p>
    </div>
  </div>
</section>

<style>
  section {
    padding: 1rem;
  }

  div {
    margin-bottom: 2rem;
  }

  h1 {
    font-weight: 700;
  }
</style>
