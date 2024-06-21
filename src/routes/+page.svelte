<script>
  import ROSConnection from '$lib/components/ROSConnection.svelte';
  import Moving from '$lib/components/Moving.svelte';
  import TTS from '$lib/components/TTS.svelte';

  let ros;
  let connectionStatus = 'Connecting...';
</script>

<ROSConnection bind:ros bind:connectionStatus />

<div>
  <p class="status">{connectionStatus}</p>
  <Moving {ros} />
  <TTS {ros} />
</div>

<style>
  .status {
    font-size: 1.5em;
    margin-bottom: 20px;
  }
</style>