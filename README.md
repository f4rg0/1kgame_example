# 1k Game Example - https://furucode.github.io/1kgame_example/

This game is made in 1kb or under when it comes to the rules of 1kb = 1024b. Feel free to play this game, look at the source code and change it in any way that you like. May God bless you all.
// Translate to flutter, nice job BigK
import 'dart:async';
import 'dart:math';
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        backgroundColor: const Color(0xFF228822),
        body: GameScreen(),
      ),
    );
  }
}

class GameScreen extends StatefulWidget {
  @override
  _GameScreenState createState() => _GameScreenState();
}

class _GameScreenState extends State<GameScreen> {
  double playerX = 144.0; // Player's X position
  int score = 0;
  bool playing = true;
  final List<Rect> fallingObjects = [];
  final Random random = Random();

  late Timer gameTimer;

  @override
  void initState() {
    super.initState();
    // Initialize falling objects
    for (int i = 0; i < 5; i++) {
      fallingObjects.add(_generateNewFallingObject());
    }

    // Start the game loop
    gameTimer = Timer.periodic(const Duration(milliseconds: 16), (timer) {
      _updateGame();
    });
  }

  @override
  void dispose() {
    gameTimer.cancel();
    super.dispose();
  }

  void _updateGame() {
    setState(() {
      // Update falling objects
      for (int i = 0; i < fallingObjects.length; i++) {
        Rect obj = fallingObjects[i];
        fallingObjects[i] = obj.translate(0, 2); // Move down

        // Check if object went off screen
        if (fallingObjects[i].top > 320) {
          fallingObjects[i] = _generateNewFallingObject();
          score += 5;
        }

        // Check for collision with player
        Rect playerRect = Rect.fromLTWH(playerX, 256, 32, 32);
        if (fallingObjects[i].overlaps(playerRect)) {
          score -= 1;
        }
      }

      // Move player
      if (playing) {
        playerX = (playerX - 2).clamp(0.0, 288.0);
      } else {
        playerX = (playerX + 2).clamp(0.0, 288.0);
      }
    });
  }

  Rect _generateNewFallingObject() {
    return Rect.fromLTWH(random.nextDouble() * 289, -24, 24, 24);
  }

  void _toggleGame() {
    setState(() {
      playing = !playing;
    });
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _toggleGame,
      child: CustomPaint(
        size: const Size(320, 320),
        painter: GamePainter(playerX: playerX, score: score, fallingObjects: fallingObjects),
      ),
    );
  }
}

class GamePainter extends CustomPainter {
  final double playerX;
  final int score;
  final List<Rect> fallingObjects;

  GamePainter({required this.playerX, required this.score, required this.fallingObjects});

  @override
  void paint(Canvas canvas, Size size) {
    final Paint paint = Paint();

    // Draw background
    paint.color = const Color(0xFF88F);
    canvas.drawRect(Rect.fromLTWH(0, 0, size.width, size.height), paint);

    // Draw ground
    paint.color = Colors.green;
    canvas.drawRect(Rect.fromLTWH(0, 288, size.width, 32), paint);

    // Draw player
    paint.color = Colors.black;
    canvas.drawRect(Rect.fromLTWH(playerX, 256, 32, 32), paint);

    // Draw falling objects
    paint.color = Colors.red;
    for (Rect obj in fallingObjects) {
      canvas.drawRect(obj, paint);
    }

    // Draw score
    final textPainter = TextPainter(
      text: TextSpan(
        text: '$score',
        style: const TextStyle(color: Colors.white, fontSize: 32),
      ),
      textAlign: TextAlign.left,
      textDirection: TextDirection.ltr,
    );
    textPainter.layout();
    textPainter.paint(canvas, const Offset(4, 4));
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => true;
}
