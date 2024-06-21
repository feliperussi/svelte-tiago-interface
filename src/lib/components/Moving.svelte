<script>
    import ROSLIB from 'roslib';
  
    export let ros;
    let linearX = 0;
    let angularZ = 0;
    let intervalId;
  
    function sendCmdVel() {
      const cmdVelTopic = new ROSLIB.Topic({
        ros: ros,
        name: '/input_joy/cmd_vel',
        messageType: 'geometry_msgs/Twist'
      });
  
      const twist = new ROSLIB.Message({
        linear: {
          x: linearX,
          y: 0,
          z: 0
        },
        angular: {
          x: 0,
          y: 0,
          z: angularZ
        }
      });
  
      cmdVelTopic.publish(twist);
    }
  
    function handlePressStart(direction) {
      switch (direction) {
        case 'up':
          linearX = 1;
          break;
        case 'down':
          linearX = -1;
          break;
        case 'left':
          angularZ = 1;
          break;
        case 'right':
          angularZ = -1;
          break;
      }
      sendCmdVel();
      intervalId = setInterval(sendCmdVel, 10);
    }
  
    function handlePressEnd() {
      clearInterval(intervalId);
      linearX = 0;
      angularZ = 0;
      sendCmdVel();
    }
  </script>
  
  <style>
    .keypad-container {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    .keypad-row {
      display: flex;
      justify-content: center;
      margin: 5px 0;
    }
    .key {
      width: 50px;
      height: 50px;
      margin: 5px;
      border: 1px solid #000;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      user-select: none;
    }
    .key:active {
      background-color: #ccc;
    }
  </style>
  
  <div class="keypad-container">
    <div class="keypad-row">
      <div class="key" 
        on:mousedown={() => handlePressStart('up')} 
        on:mouseup={handlePressEnd} 
        on:touchstart={() => handlePressStart('up')} 
        on:touchend={handlePressEnd}>▲
      </div>
    </div>
    <div class="keypad-row">
      <div class="key" 
        on:mousedown={() => handlePressStart('left')} 
        on:mouseup={handlePressEnd} 
        on:touchstart={() => handlePressStart('left')} 
        on:touchend={handlePressEnd}>◄
      </div>
      <div class="key" 
        on:mousedown={() => handlePressStart('down')} 
        on:mouseup={handlePressEnd} 
        on:touchstart={() => handlePressStart('down')} 
        on:touchend={handlePressEnd}>▼
      </div>
      <div class="key" 
        on:mousedown={() => handlePressStart('right')} 
        on:mouseup={handlePressEnd} 
        on:touchstart={() => handlePressStart('right')} 
        on:touchend={handlePressEnd}>►
      </div>
    </div>
  </div>