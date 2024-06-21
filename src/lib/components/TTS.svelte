<script>
    import ROSLIB from 'roslib';
    export let ros;
    let ttsText = '';
    let ttsFeedback = '';
    let ttsResult = '';
  
    function sendTTSMessage() {
      const ttsClient = new ROSLIB.ActionClient({
        ros: ros,
        serverName: '/tts',
        actionName: 'pal_interaction_msgs/TtsAction'
      });
  
      const goal = new ROSLIB.Goal({
        actionClient: ttsClient,
        goalMessage: {
          rawtext: {
            text: ttsText,
            lang_id: 'en_US'
          }
        }
      });
  
      goal.on('feedback', (feedback) => {
        ttsFeedback = JSON.stringify(feedback);
        console.log('Feedback: ', feedback);
      });
  
      goal.on('result', (result) => {
        ttsResult = JSON.stringify(result);
        console.log('Result: ', result);
      });
  
      goal.send();
    }
  </script>
  
  <style>
    .tts-container {
      margin-top: 20px;
    }
    input {
      padding: 5px;
      margin-right: 10px;
    }
    button {
      padding: 5px 10px;
    }
  </style>
  
  <div class="tts-container">
    <input type="text" bind:value={ttsText} placeholder="Enter TTS text" />
    <button on:click={sendTTSMessage}>Send TTS</button>
    <p>{ttsFeedback}</p>
    <p>{ttsResult}</p>
  </div>