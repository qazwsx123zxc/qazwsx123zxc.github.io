<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>低功耗舵机控制器 - 充电宝供电优化</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }
        body {
            font-family: 'PingFang SC', 'Helvetica Neue', Arial, sans-serif;
            background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
            color: #fff;
            min-height: 100vh;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
        .container {
            width: 100%;
            max-width: 500px;
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 25px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
            text-align: center;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        .header {
            margin-bottom: 20px;
        }
        h1 {
            font-size: 2rem;
            margin-bottom: 10px;
            text-shadow: 0 2px 4px rgba(0,0,0,0.2);
            color: #4fc3f7;
        }
        .subtitle {
            font-size: 0.9rem;
            color: rgba(255, 255, 255, 0.7);
            margin-bottom: 5px;
        }
        .power-info {
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 15px 0;
            padding: 10px;
            background: rgba(0, 0, 0, 0.2);
            border-radius: 15px;
        }
        .battery-icon {
            width: 30px;
            height: 15px;
            border: 2px solid #4fc3f7;
            border-radius: 4px;
            margin-right: 10px;
            position: relative;
        }
        .battery-icon::after {
            content: '';
            position: absolute;
            width: 4px;
            height: 7px;
            background: #4fc3f7;
            top: 3px;
            right: -6px;
            border-radius: 0 2px 2px 0;
        }
        .battery-level {
            width: 80%;
            height: 70%;
            background: #4fc3f7;
            margin: 2px;
            border-radius: 2px;
        }
        
        /* 控制面板 */
        .control-panel {
            background: rgba(0, 0, 0, 0.2);
            border-radius: 18px;
            padding: 20px;
            margin: 15px 0;
            position: relative;
            overflow: hidden;
        }
        .panel-title {
            font-size: 1.2rem;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #4fc3f7;
        }
        
        /* 舵机状态指示器 */
        .servo-indicator {
            width: 150px;
            height: 150px;
            margin: 0 auto 20px;
            position: relative;
        }
        .servo-base {
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.05);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            position: relative;
            box-shadow: inset 0 0 15px rgba(0,0,0,0.2);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        .servo-arm {
            position: absolute;
            width: 6px;
            height: 65px;
            background: #4fc3f7;
            border-radius: 3px;
            bottom: 50%;
            left: 50%;
            transform: translateX(-50%) rotate(0deg);
            transform-origin: bottom center;
            transition: transform 0.8s cubic-bezier(0.34, 1.56, 0.64, 1);
            box-shadow: 0 0 10px rgba(79, 195, 247, 0.5);
        }
        .servo-arm::after {
            content: '';
            position: absolute;
            width: 20px;
            height: 20px;
            background: #4fc3f7;
            border-radius: 50%;
            top: -10px;
            left: 50%;
            transform: translateX(-50%);
        }
        .angle-display {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 2rem;
            font-weight: bold;
            text-shadow: 0 2px 4px rgba(0,0,0,0.3);
            color: #4fc3f7;
        }
        
        /* 控制按钮 */
        .control-btn {
            background: linear-gradient(45deg, #0288d1, #4fc3f7);
            color: white;
            border: none;
            padding: 15px 30px;
            font-size: 1.2rem;
            font-weight: bold;
            border-radius: 40px;
            margin: 12px auto;
            display: block;
            width: 100%;
            max-width: 250px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
            cursor: pointer;
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
        }
        .control-btn:active {
            transform: scale(0.96);
            box-shadow: 0 2px 8px rgba(0,0,0,0.2);
        }
        .control-btn.disabled {
            background: linear-gradient(45deg, #546e7a, #78909c);
            cursor: not-allowed;
        }
        
        /* 低功耗开关 */
        .power-toggle {
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 15px 0;
        }
        .toggle-text {
            margin: 0 10px;
            font-size: 0.9rem;
        }
        .toggle-switch {
            position: relative;
            display: inline-block;
            width: 50px;
            height: 24px;
        }
        .toggle-switch input {
            opacity: 0;
            width: 0;
            height: 0;
        }
        .slider {
            position: absolute;
            cursor: pointer;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: #ccc;
            transition: .4s;
            border-radius: 24px;
        }
        .slider:before {
            position: absolute;
            content: "";
            height: 18px;
            width: 18px;
            left: 3px;
            bottom: 3px;
            background-color: white;
            transition: .4s;
            border-radius: 50%;
        }
        input:checked + .slider {
            background-color: #4fc3f7;
        }
        input:checked + .slider:before {
            transform: translateX(26px);
        }
        
        /* 状态面板 */
        .status-panel {
            background: rgba(0, 0, 0, 0.2);
            border-radius: 18px;
            padding: 18px;
            margin-top: 15px;
        }
        .status-row {
            display: flex;
            justify-content: space-between;
            margin-bottom: 12px;
            padding-bottom: 12px;
            border-bottom: 1px solid rgba(255, 255, 255, 0.05);
            font-size: 0.9rem;
        }
        .status-row:last-child {
            margin-bottom: 0;
            padding-bottom: 0;
            border-bottom: none;
        }
        .status-label {
            display: flex;
            align-items: center;
        }
        .status-value {
            font-weight: bold;
            color: #4fc3f7;
        }
        .status-indicator {
            display: inline-block;
            width: 10px;
            height: 10px;
            border-radius: 50%;
            margin-right: 8px;
        }
        .online { background-color: #4fc3f7; box-shadow: 0 0 6px #4fc3f7; }
        .offline { background-color: #f44336; box-shadow: 0 0 6px #f44336; }
        .processing { 
            background-color: #ffeb3b; 
            box-shadow: 0 0 6px #ffeb3b;
            animation: pulse 1.5s infinite;
        }
        @keyframes pulse {
            0% { opacity: 0.5; transform: scale(0.9); }
            50% { opacity: 1; transform: scale(1.1); }
            100% { opacity: 0.5; transform: scale(0.9); }
        }
        
        /* 功耗统计 */
        .power-stats {
            display: flex;
            justify-content: space-around;
            margin: 15px 0;
            text-align: center;
        }
        .stat-item {
            padding: 10px;
        }
        .stat-value {
            font-size: 1.2rem;
            font-weight: bold;
            color: #4fc3f7;
        }
        .stat-label {
            font-size: 0.8rem;
            color: rgba(255, 255, 255, 0.7);
        }
        
        /* 响应式设计 */
        @media (max-width: 600px) {
            .container {
                padding: 15px;
                border-radius: 15px;
            }
            h1 {
                font-size: 1.6rem;
            }
            .control-btn {
                padding: 12px 25px;
                font-size: 1.1rem;
            }
            .servo-indicator {
                width: 130px;
                height: 130px;
            }
        }
        
        /* 180度动画效果 */
        @keyframes rotateArm180deg {
            0% { transform: translateX(-50%) rotate(0deg); }
            50% { transform: translateX(-50%) rotate(180deg); }
            100% { transform: translateX(-50%) rotate(0deg); }
        }
        .rotating-180deg {
            animation: rotateArm180deg 2.5s ease-in-out;
        }
        
        /* 功耗模式指示 */
        .power-mode {
            padding: 8px 15px;
            border-radius: 20px;
            font-size: 0.8rem;
            margin: 10px auto;
            display: inline-block;
        }
        .power-low {
            background: rgba(79, 195, 247, 0.2);
            color: #4fc3f7;
        }
        .power-normal {
            background: rgba(255, 152, 0, 0.2);
            color: #ff9800;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>低功耗舵机控制器</h1>
            <div class="subtitle">充电宝供电优化 · 180度精确控制</div>
            
            <div class="power-info">
                <div class="battery-icon">
                    <div class="battery-level" id="batteryLevel"></div>
                </div>
                <div>电池优化模式</div>
            </div>
        </div>
        
        <div class="control-panel">
            <div class="panel-title">
                <i>⚙️</i> 180度精确控制
            </div>
            
            <div class="servo-indicator">
                <div class="servo-base">
                    <div class="servo-arm" id="servoArm"></div>
                    <div class="angle-display" id="angleDisplay">0°</div>
                </div>
            </div>
            
            <button class="control-btn" id="controlBtn">转动舵机180度</button>
            
            <div class="power-mode power-low" id="powerMode">低功耗模式已启用</div>
            
            <div class="power-toggle">
                <span class="toggle-text">低功耗模式</span>
                <label class="toggle-switch">
                    <input type="checkbox" id="powerToggle" checked>
                    <span class="slider"></span>
                </label>
            </div>
        </div>
        
        <div class="power-stats">
            <div class="stat-item">
                <div class="stat-value" id="powerSaved">~65%</div>
                <div class="stat-label">功耗节省</div>
            </div>
            <div class="stat-item">
                <div class="stat-value" id="batteryLife">+8h</div>
                <div class="stat-label">续航提升</div>
            </div>
            <div class="stat-item">
                <div class="stat-value" id="activeTime">0s</div>
                <div class="stat-label">活动时间</div>
            </div>
        </div>
        
        <div class="status-panel">
            <div class="status-row">
                <div class="status-label">
                    <span class="status-indicator" id="deviceIndicator"></span>
                    设备状态:
                </div>
                <div class="status-value" id="deviceStatus">检测中...</div>
            </div>
            
            <div class="status-row">
                <div class="status-label">
                    <span class="status-indicator" id="actionIndicator"></span>
                    操作状态:
                </div>
                <div class="status-value" id="actionStatus">就绪</div>
            </div>
            
            <div class="status-row">
                <div class="status-label">WiFi信号:</div>
                <div class="status-value" id="wifiStatus">-- dBm</div>
            </div>
            
            <div class="status-row">
                <div class="status-label">运行时间:</div>
                <div class="status-value" id="uptime">--</div>
            </div>
        </div>
    </div>

    <script>
    // 设备状态元素
    const deviceIndicator = document.getElementById('deviceIndicator');
    const deviceStatus = document.getElementById('deviceStatus');
    const actionIndicator = document.getElementById('actionIndicator');
    const actionStatus = document.getElementById('actionStatus');
    const wifiStatus = document.getElementById('wifiStatus');
    const historyList = document.getElementById('historyList');
    const controlBtn = document.getElementById('controlBtn');
    const servoArm = document.getElementById('servoArm');
    const angleDisplay = document.getElementById('angleDisplay');
    const powerToggle = document.getElementById('powerToggle');
    const powerMode = document.getElementById('powerMode');
    const powerSaved = document.getElementById('powerSaved');
    const batteryLife = document.getElementById('batteryLife');
    const activeTime = document.getElementById('activeTime');
    const uptime = document.getElementById('uptime');
    const batteryLevel = document.getElementById('batteryLevel');
    
    // 设备状态常量
    const DEVICE_STATUS = {
        CHECKING: { text: '检测中...', class: 'processing' },
        ONLINE: { text: '在线 ✅', class: 'online' },
        OFFLINE: { text: '离线 ❌', class: 'offline' },
        ERROR: { text: '检测失败', class: 'offline' }
    };
    
    // 操作状态常量
    const ACTION_STATUS = {
        READY: { text: '就绪', class: 'online' },
        SENDING: { text: '发送指令中...', class: 'processing' },
        SENT: { text: '指令已发送!', class: 'online' },
        FAILED: { text: '发送失败', class: 'offline' },
        NETWORK_ERROR: { text: '网络错误', class: 'offline' }
    };
    
    // Blynk配置信息
    const BLYNK_AUTH_TOKEN = "0kEVzmxEOib9_9yFgyKkNEg_TWP8u6eB";
    const CONTROL_PIN = "V0"; // 控制舵机的虚拟引脚
    const POWER_PIN = "V1";   // 控制低功耗模式的虚拟引脚
    
    // 功耗统计
    let activeSeconds = 0;
    let powerSavingMode = true;
    
    // 更新状态显示
    function updateStatus(element, indicator, statusObj) {
        element.textContent = statusObj.text;
        indicator.className = 'status-indicator ' + statusObj.class;
    }
    
    // 控制舵机转动180度
    function controlServo() {
        // 更新状态
        updateStatus(actionStatus, actionIndicator, ACTION_STATUS.SENDING);
        controlBtn.disabled = true;
        controlBtn.classList.add('disabled');
        
        // 开始舵机动画
        servoArm.classList.remove('rotating-180deg');
        setTimeout(() => {
            servoArm.classList.add('rotating-180deg');
            angleDisplay.textContent = "180°";
        }, 50);
        
        // 使用Blynk官方API发送转动指令
        fetch(`https://blynk.cloud/external/api/update?token=${BLYNK_AUTH_TOKEN}&${CONTROL_PIN}=1`)
            .then(response => {
                if(response.ok) {
                    updateStatus(actionStatus, actionIndicator, ACTION_STATUS.SENT);
                    
                    // 3秒后重置状态
                    setTimeout(() => {
                        updateStatus(actionStatus, actionIndicator, ACTION_STATUS.READY);
                        controlBtn.disabled = false;
                        controlBtn.classList.remove('disabled');
                        angleDisplay.textContent = "0°";
                    }, 3000);
                } else {
                    updateStatus(actionStatus, actionIndicator, ACTION_STATUS.FAILED);
                    controlBtn.disabled = false;
                    controlBtn.classList.remove('disabled');
                    angleDisplay.textContent = "0°";
                }
            })
            .catch(error => {
                updateStatus(actionStatus, actionIndicator, ACTION_STATUS.NETWORK_ERROR);
                controlBtn.disabled = false;
                controlBtn.classList.remove('disabled');
                angleDisplay.textContent = "0°";
            });
    }
    
    // 切换低功耗模式
    function togglePowerMode() {
        powerSavingMode = powerToggle.checked;
        
        if (powerSavingMode) {
            powerMode.textContent = "低功耗模式已启用";
            powerMode.className = "power-mode power-low";
        } else {
            powerMode.textContent = "正常模式";
            powerMode.className = "power-mode power-normal";
        }
        
        // 发送模式切换指令
        fetch(`https://blynk.cloud/external/api/update?token=${BLYNK_AUTH_TOKEN}&${POWER_PIN}=${powerSavingMode ? 1 : 0}`)
            .catch(error => console.error('模式切换失败'));
    }
    
    // 获取设备状态
    function checkDeviceStatus() {
        fetch(`https://blynk.cloud/external/api/isHardwareConnected?token=${BLYNK_AUTH_TOKEN}`)
            .then(response => response.text())
            .then(data => {
                if(data === 'true') {
                    updateStatus(deviceStatus, deviceIndicator, DEVICE_STATUS.ONLINE);
                    
                    // 获取WiFi信号强度
                    fetch(`https://blynk.cloud/external/api/get?token=${BLYNK_AUTH_TOKEN}&V2`)
                        .then(response => response.text())
                        .then(uptimeValue => {
                            uptime.textContent = formatUptime(uptimeValue);
                        });
                } else {
                    updateStatus(deviceStatus, deviceIndicator, DEVICE_STATUS.OFFLINE);
                }
            })
            .catch(error => {
                updateStatus(deviceStatus, deviceIndicator, DEVICE_STATUS.ERROR);
            });
    }
    
    // 格式化运行时间
    function formatUptime(seconds) {
        if (!seconds) return "--";
        const hrs = Math.floor(seconds / 3600);
        const mins = Math.floor((seconds % 3600) / 60);
        const secs = seconds % 60;
        return `${hrs}h ${mins}m ${secs}s`;
    }
    
    // 模拟电池状态更新
    function updateBatteryStatus() {
        // 在真实应用中，这里可以从设备获取电池状态
        // 这里使用模拟值
        const levels = [70, 75, 80, 85, 90, 95, 100];
        const randomLevel = levels[Math.floor(Math.random() * levels.length)];
        batteryLevel.style.width = randomLevel + '%';
    }
    
    // 页面加载时初始化
    window.onload = function() {
        // 初始状态
        updateStatus(deviceStatus, deviceIndicator, DEVICE_STATUS.CHECKING);
        updateStatus(actionStatus, actionIndicator, ACTION_STATUS.READY);
        
        // 立即检查设备状态
        checkDeviceStatus();
        
        // 每30秒检查一次设备状态
        setInterval(checkDeviceStatus, 30000);
        
        // 每10秒更新电池状态
        setInterval(updateBatteryStatus, 10000);
        updateBatteryStatus();
        
        // 绑定按钮事件
        controlBtn.addEventListener('click', controlServo);
        powerToggle.addEventListener('change', togglePowerMode);
        
        // 更新活动时间
        setInterval(() => {
            activeSeconds++;
            activeTime.textContent = activeSeconds + 's';
        }, 1000);
    };
    </script>
</body>
</html>
