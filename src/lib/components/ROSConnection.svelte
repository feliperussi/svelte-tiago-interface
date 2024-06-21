<script>
    import { onMount } from 'svelte';
    import ROSLIB from 'roslib';
  
    export let ros;
    export let connectionStatus;
  
    onMount(() => {
      ros = new ROSLIB.Ros({
        url: 'ws://10.68.0.1:9090'  // Replace with your robot's IP
      });
  
      ros.on('connection', () => {
        console.log('Connected to websocket server.');
        connectionStatus = 'Connected';
      });
  
      ros.on('error', (error) => {
        console.log('Error connecting to websocket server: ', error);
        connectionStatus = 'Error';
      });
  
      ros.on('close', () => {
        console.log('Connection to websocket server closed.');
        connectionStatus = 'Closed';
      });
    });
  </script>