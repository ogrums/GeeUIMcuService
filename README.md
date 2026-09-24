# GeeUIMcuService

System app that owns the MCU serial port. It turns robot commands into AT strings and turns MCU sensor lines back into AIDL callbacks. It does not use `RobotSdk`.

## Package

- `com.letianpai.robot.mcuservice`
- system uid
- `LTPMcuService` action: `android.intent.action.LTPMCU`
- Sensor service action: `android.intent.action.geeui.SENSOR`

Submodules: `GeeUIBase` (`library`, `CommandLib`) and `GeeUIComponets`. `settings.gradle` spells the second path `GeeUIComponents`. `.gitmodules` only lists `GeeUIComponets`.

## What it does

Native code is `SerialAllJNI` (`libSerialAllLib`, CMake under `app/src/main/cpp`): `openPort`, `closePort`, `writeData`, `registerSensorDataListener`. The launcher copy of this service opens `/dev/ttyS5`.

`GeeUISensorsService` parses MCU lines `AT+INT,...` and `AT+RES,...`, including `AT+Gsys` and `AT+VerR`. It reports light, touch, cliff, suspend, waggle, fall-down, IR, and time-of-flight. A bootloader response starts `com.letianpai.otaservice.L81OtaActivity`. A comment notes that IR error grows as temperature rises.

`LTPMcuService` binds:

- `com.renhejia.robot.letianpaiservice` / `android.intent.action.LETIANPAI`, and registers `LtpMcuCommandCallback`
- the local `GeeUISensorsService`

Incoming motion commands go to `com.letianpai.McuCommandControlManager.commandDistribute`, which writes `AT+MOVEW`, `AT+EARW`, `AT+LEDOn`, `AT+LEDOff`, `AT+FunCtr`, `AT+Reset`, and `AT+FiAGW`. Walk directions in the comments are forward, back, left, and right.

Sensor policy calls `ILetianpaiService.setSensorResponse` for tap, double-tap, long-press, fall down, precipice, cliff dodge (`fallBackend`, `fallForward`, `fallLeft`, `fallRight`), TOF obstacle, and shake.

Exported AIDL `ISensorService`: `writeAtCommand`, plus register and unregister for write results and IR.

`com.letianpai.robot.mcuservice.manager.McuCommandControlManager` is fully commented out. The live class is `com.letianpai.McuCommandControlManager`. `McuControlManager` is an in-process walker for motors 1–6 (feet, legs, ears) and is not what `LTPMcuService` calls.

## Comment glossary

| Where | Chinese | English |
|---|---|---|
| `LTPMcuService` | 打开串口 / 关闭串口 | Open / close the serial port |
| `LTPMcuService` | 乐天派 MCU 完成AIDLService服务 | MCU finished binding the AIDL service |
| `LTPMcuService` | 乐天派 MCU 无法绑定aidlserver的AIDLService服务 | MCU could not bind the AIDL server |
| `LTPMcuService` | 往后走 / 往前走 / 往右走 / 往左走 | Walk back / forward / right / left |
| `LTPMcuService` | 随着温度的升高，红外误差会加大 | IR error grows as temperature rises |
| `MCUConsts` | 读取悬崖传感器状态 | Read cliff-sensor status (the same comment is reused on several AT strings) |
| `MCUCommandConsts` | 天线控制 / 向前 | Antenna control / forward |
| `McuCommandControlManager` | 1 环境光 … 6 陀螺仪 | `AT+FunCtr`: 1 ambient light, 2 touch, 3 leg and foot servo power, 4 ear light, 5 cliff and suspend, 6 gyro |
