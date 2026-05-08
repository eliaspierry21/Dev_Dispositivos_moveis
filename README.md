nível 1

import 'package:flutter/material.dart';
import 'package:permission_handler/permission_handler.dart';

void main() => runApp(MaterialApp(home: CameraPermissionScreen()));

class CameraPermissionScreen extends StatefulWidget {
  @override
  _CameraPermissionScreenState createState() => _CameraPermissionScreenState();
}

class _CameraPermissionScreenState extends State<CameraPermissionScreen> {
  String _status = "Aguardando solicitação...";

  Future<void> _requestCameraPermission() async {
    // Solicita a permissão
    var status = await Permission.camera.request();

    setState(() {
      if (status.isGranted) {
        _status = "Permissão concedida";
      } else if (status.isDenied) {
        _status = "Permissão negada";
      } else if (status.isPermanentlyDenied) {
        _status = "Permissão negada permanentemente (ajuste nas configurações)";
        openAppSettings(); // Opcional: abre as configurações do celular
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Nível 1")),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text("Nível 1 — Solicitação de Permissão", 
                 style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            SizedBox(height: 20),
            Text("Requisito:", style: TextStyle(fontWeight: FontWeight.bold)),
            Text("Solicitar permissão da câmera."),
            SizedBox(height: 20),
            Text("Esperado:", style: TextStyle(fontWeight: FontWeight.bold)),
            Text("• $_status", 
                 style: TextStyle(color: _status.contains("concedida") ? Colors.green : Colors.red)),
            Spacer(),
            Center(
              child: ElevatedButton(
                onPressed: _requestCameraPermission,
                child: Text("Solicitar Permissão"),
              ),
            ),
          ],
        ),
      ),
    );
  }
}


nível 2

import 'dart:async';
import 'package:flutter/material.dart';
import 'package:sensors_plus/sensors_plus.dart';

void main() => runApp(MaterialApp(home: AccelerometerScreen()));

class AccelerometerScreen extends StatefulWidget {
  @override
  _AccelerometerScreenState createState() => _AccelerometerScreenState();
}

class _AccelerometerScreenState extends State<AccelerometerScreen> {
  // Variáveis para armazenar os eixos
  double x = 0, y = 0, z = 0;
  
  // Subscription para gerenciar a escuta do sensor
  StreamSubscription<AccelerometerEvent>? _subscription;

  @override
  void initState() {
    super.initState();
    // Inicia a escuta do acelerômetro
    _subscription = accelerometerEvents.listen((AccelerometerEvent event) {
      setState(() {
        x = event.x;
        y = event.y;
        z = event.z;
      });
    });
  }

  @override
  void dispose() {
    // É vital cancelar o subscription para evitar vazamento de memória
    _subscription?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Nível 2")),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text("Nível 2 — Captura do acelerômetro", 
                 style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            SizedBox(height: 30),
            Text("Requisito:", style: TextStyle(fontWeight: FontWeight.bold)),
            Text("Mostrar valores X, Y e Z na tela."),
            SizedBox(height: 40),
            
            // Exibição dos Valores
            _buildSensorRow("Eixo X:", x),
            _buildSensorRow("Eixo Y:", y),
            _buildSensorRow("Eixo Z:", z),
            
            Spacer(),
            Center(
              child: Text(
                "Mova o dispositivo para ver a alteração",
                style: TextStyle(color: Colors.grey, fontStyle: FontStyle.italic),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildSensorRow(String label, double value) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8.0),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(label, style: TextStyle(fontSize: 20, fontWeight: FontWeight.w500)),
          Text(value.toStringAsFixed(2), // Limita a 2 casas decimais
               style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold, color: Colors.blueAccent)),
        ],
      ),
    );
  }
}


nível 3

import 'dart:async';
import 'package:flutter/material.dart';
import 'package:sensors_plus/sensors_plus.dart';

void main() => runApp(MaterialApp(home: ShakeDetectionScreen()));

class ShakeDetectionScreen extends StatefulWidget {
  @override
  _ShakeDetectionScreenState createState() => _ShakeDetectionScreenState();
}

class _ShakeDetectionScreenState extends State<ShakeDetectionScreen> {
  double _xAxis = 0;
  bool _isMovementDetected = false;
  StreamSubscription<AccelerometerEvent>? _subscription;
  Timer? _displayTimer;

  @override
  void initState() {
    super.initState();
    // Escutando os eventos do acelerômetro
    _subscription = accelerometerEvents.listen((AccelerometerEvent event) {
      setState(() {
        _xAxis = event.x;
        
        // Regra do Nível 3: event.x > 8
        // Usamos .abs() para detectar o movimento brusco em ambas as direções do eixo X
        if (_xAxis.abs() > 8) {
          _triggerDetection();
        }
      });
    });
  }

  void _triggerDetection() {
    setState(() {
      _isMovementDetected = true;
    });

    // Reseta o aviso após 1.5 segundos sem movimento brusco
    _displayTimer?.cancel();
    _displayTimer = Timer(Duration(milliseconds: 1500), () {
      setState(() {
        _isMovementDetected = false;
      });
    });
  }

  @override
  void dispose() {
    _subscription?.cancel();
    _displayTimer?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Nível 3")),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(20.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text("Nível 3 — Detectar movimento brusco", 
                   style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
              SizedBox(height: 40),
              
              Text("Valor atual de X:", style: TextStyle(color: Colors.grey)),
              Text(_xAxis.toStringAsFixed(2), 
                   style: TextStyle(fontSize: 48, fontWeight: FontWeight.w300)),
              
              SizedBox(height: 60),
              
              // Área de Alerta
              AnimatedContainer(
                duration: Duration(milliseconds: 300),
                padding: EdgeInsets.symmetric(horizontal: 30, vertical: 20),
                decoration: BoxDecoration(
                  color: _isMovementDetected ? Colors.redAccent : Colors.transparent,
                  borderRadius: BorderRadius.circular(10),
                  border: Border.all(
                    color: _isMovementDetected ? Colors.red : Colors.grey.shade300
                  )
                ),
                child: Text(
                  _isMovementDetected ? "Movimento detectado" : "Aguardando movimento...",
                  style: TextStyle(
                    fontSize: 22, 
                    fontWeight: FontWeight.bold,
                    color: _isMovementDetected ? Colors.white : Colors.grey
                  ),
                ),
              ),
              
              SizedBox(height: 20),
              Text("Regra: Se event.x > 8", 
                   style: TextStyle(fontStyle: FontStyle.italic, color: Colors.blueGrey)),
            ],
          ),
        ),
      ),
    );
  }
}

nível 4
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:sensors_plus/sensors_plus.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

void main() => runApp(MaterialApp(home: NotificationLevelScreen()));

class NotificationLevelScreen extends StatefulWidget {
  @override
  _NotificationLevelScreenState createState() => _NotificationLevelScreenState();
}

class _NotificationLevelScreenState extends State<NotificationLevelScreen> {
  final FlutterLocalNotificationsPlugin _notificationsPlugin = FlutterLocalNotificationsPlugin();
  StreamSubscription<AccelerometerEvent>? _subscription;
  bool _canSendNotification = true; // Para evitar spam de notificações

  @override
  void initState() {
    super.initState();
    _setupNotifications();
    _startSensor();
  }

  // Inicialização das notificações
  Future<void> _setupNotifications() async {
    const AndroidInitializationSettings initializationSettingsAndroid =
        AndroidInitializationSettings('@mipmap/ic_launcher');
    
    const InitializationSettings initializationSettings = InitializationSettings(
      android: initializationSettingsAndroid,
    );

    await _notificationsPlugin.initialize(initializationSettings);
  }

  void _startSensor() {
    _subscription = accelerometerEvents.listen((AccelerometerEvent event) {
      // Regra de movimento (X > 8)
      if (event.x.abs() > 8 && _canSendNotification) {
        _sendLocalNotification();
      }
    });
  }

  Future<void> _sendLocalNotification() async {
    _canSendNotification = false; // Bloqueia temporariamente para não travar o celular

    const AndroidNotificationDetails androidDetails = AndroidNotificationDetails(
      'movimento_id',
      'Alertas de Movimento',
      importance: Importance.max,
      priority: Priority.high,
    );

    const NotificationDetails platformDetails = NotificationDetails(android: androidDetails);

    await _notificationsPlugin.show(
      0,
      'Movimento Detectado!',
      'O acelerômetro registrou uma alteração brusca.',
      platformDetails,
    );

    // Espera 5 segundos antes de permitir outra notificação
    Timer(Duration(seconds: 5), () {
      if (mounted) setState(() => _canSendNotification = true);
    });
  }

  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Nível 4")),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text("Nível 4 — Notificação local", 
                 style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            SizedBox(height: 20),
            Text("Requisito:", style: TextStyle(fontWeight: FontWeight.bold)),
            Text("Quando houver movimento: enviar notificação local."),
            SizedBox(height: 40),
            Center(
              child: Column(
                children: [
                  Icon(
                    _canSendNotification ? Icons.notifications_active : Icons.notifications_off,
                    size: 80,
                    color: _canSendNotification ? Colors.blue : Colors.grey,
                  ),
                  SizedBox(height: 20),
                  Text(
                    _canSendNotification 
                      ? "Balance o telemóvel para testar" 
                      : "Notificação enviada! Aguarde...",
                    textAlign: TextAlign.center,
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}


nível 5

import 'dart:async';
import 'package:flutter/material.dart';
import 'package:sensors_plus/sensors_plus.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

void main() => runApp(MaterialApp(home: NotificationLevelScreen()));

class NotificationLevelScreen extends StatefulWidget {
  @override
  _NotificationLevelScreenState createState() => _NotificationLevelScreenState();
}

class _NotificationLevelScreenState extends State<NotificationLevelScreen> {
  final FlutterLocalNotificationsPlugin _notificationsPlugin = FlutterLocalNotificationsPlugin();
  StreamSubscription<AccelerometerEvent>? _subscription;
  bool _canSendNotification = true; // Para evitar spam de notificações

  @override
  void initState() {
    super.initState();
    _setupNotifications();
    _startSensor();
  }

  // Inicialização das notificações
  Future<void> _setupNotifications() async {
    const AndroidInitializationSettings initializationSettingsAndroid =
        AndroidInitializationSettings('@mipmap/ic_launcher');
    
    const InitializationSettings initializationSettings = InitializationSettings(
      android: initializationSettingsAndroid,
    );

    await _notificationsPlugin.initialize(initializationSettings);
  }

  void _startSensor() {
    _subscription = accelerometerEvents.listen((AccelerometerEvent event) {
      // Regra de movimento (X > 8)
      if (event.x.abs() > 8 && _canSendNotification) {
        _sendLocalNotification();
      }
    });
  }

  Future<void> _sendLocalNotification() async {
    _canSendNotification = false; // Bloqueia temporariamente para não travar o celular

    const AndroidNotificationDetails androidDetails = AndroidNotificationDetails(
      'movimento_id',
      'Alertas de Movimento',
      importance: Importance.max,
      priority: Priority.high,
    );

    const NotificationDetails platformDetails = NotificationDetails(android: androidDetails);

    await _notificationsPlugin.show(
      0,
      'Movimento Detectado!',
      'O acelerômetro registrou uma alteração brusca.',
      platformDetails,
    );

    // Espera 5 segundos antes de permitir outra notificação
    Timer(Duration(seconds: 5), () {
      if (mounted) setState(() => _canSendNotification = true);
    });
  }

  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Nível 4")),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text("Nível 4 — Notificação local", 
                 style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            SizedBox(height: 20),
            Text("Requisito:", style: TextStyle(fontWeight: FontWeight.bold)),
            Text("Quando houver movimento: enviar notificação local."),
            SizedBox(height: 40),
            Center(
              child: Column(
                children: [
                  Icon(
                    _canSendNotification ? Icons.notifications_active : Icons.notifications_off,
                    size: 80,
                    color: _canSendNotification ? Colors.blue : Colors.grey,
                  ),
                  SizedBox(height: 20),
                  Text(
                    _canSendNotification 
                      ? "Balance o telemóvel para testar" 
                      : "Notificação enviada! Aguarde...",
                    textAlign: TextAlign.center,
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

nível 6

import 'dart:async';
import 'package:flutter/material.dart';
import 'package:sensors_plus/sensors_plus.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:vibration/vibration.dart';
import 'package:intl/intl.dart';

void main() => runApp(MaterialApp(home: AntiTheftSystem()));

class AntiTheftSystem extends StatefulWidget {
  @override
  _AntiTheftSystemState createState() => _AntiTheftSystemState();
}

class _AntiTheftSystemState extends State<AntiTheftSystem> {
  final FlutterLocalNotificationsPlugin _notificationsPlugin = FlutterLocalNotificationsPlugin();
  StreamSubscription<AccelerometerEvent>? _subscription;
  
  bool _isArmed = false;
  List<String> _eventLog = []; // Registo de horários

  @override
  void initState() {
    super.initState();
    _initNotifications();
  }

  void _initNotifications() async {
    const settings = InitializationSettings(android: AndroidInitializationSettings('@mipmap/ic_launcher'));
    await _notificationsPlugin.initialize(settings);
  }

  void _toggleSystem(bool value) {
    setState(() => _isArmed = value);
    if (_isArmed) {
      _startMonitoring();
    } else {
      _subscription?.cancel();
    }
  }

  void _startMonitoring() {
    _subscription = accelerometerEvents.listen((event) {
      // Regra de detecção de movimento brusco
      if (event.x.abs() > 8 || event.y.abs() > 8 || event.z.abs() > 12) {
        _triggerAlarm();
      }
    });
  }

  void _triggerAlarm() async {
    // 1. Regista o horário do evento
    String timestamp = DateFormat('HH:mm:ss - dd/MM').format(DateTime.now());
    
    setState(() {
      _eventLog.insert(0, "Movimento em: $timestamp");
      _isArmed = false; // Desarma após o evento para evitar loops
    });
    _subscription?.cancel();

    // 2. Emite Notificação Local
    const details = AndroidNotificationDetails('anti_furto', 'Alerta Seguranca', importance: Importance.max);
    await _notificationsPlugin.show(1, 'ALERTA ANTI-FURTO', 'O seu dispositivo foi movimentado!', const NotificationDetails(android: details));

    // 3. Vibra o dispositivo
    if (await Vibration.hasVibrator() ?? false) {
      Vibration.vibrate(duration: 2000); // Vibra por 2 segundos
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const
