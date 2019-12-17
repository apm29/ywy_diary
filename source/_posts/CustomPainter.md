---
title: CustomPaint
date: 2019-12-11 21:11:20
tags: 
 - Flutter
 - Dart 
---
# CustomPaint
The interface used by [CustomPaint] (in the widgets library) and
[RenderCustomPaint] (in the rendering library).
To implement a custom painter, either subclass or implement this interface
to define your custom paint delegate. [CustomPaint] subclasses must
implement the [paint] and [shouldRepaint] methods, and may optionally also
implement the [hitTest] and [shouldRebuildSemantics] methods, and the
[semanticsBuilder] getter.
The [paint] method is called whenever the custom object needs to be repainted.
The [shouldRepaint] method is called when a new instance of the class
is provided, to check if the new instance actually represents different
information.
The most efficient way to trigger a repaint is to either:
* Extend this class and supply a `repaint` argument to the constructor of
  the [CustomPainter], where that object notifies its listeners when it is
  time to repaint.
* Extend [Listenable] (e.g. via [ChangeNotifier]) and implement
  [CustomPainter], so that the object itself provides the notifications
  directly.
In either case, the [CustomPaint] widget or [RenderCustomPaint]
render object will listen to the [Listenable] and repaint whenever the
animation ticks, avoiding both the build and layout phases of the pipeline.
The [hitTest] method is called when the user interacts with the underlying
render object, to determine if the user hit the object or missed it.
The [semanticsBuilder] is called whenever the custom object needs to rebuild
its semantics information.
The [shouldRebuildSemantics] method is called when a new instance of the
class is provided, to check if the new instance contains different
information that affects the semantics tree.
## Sample code
This sample extends the same code shown for [RadialGradient] to create a
custom painter that paints a sky.
```dart
class Sky extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    var rect = Offset.zero & size;
    var gradient = new RadialGradient(
      center: const Alignment(0.7, -0.6),
      radius: 0.2,
      colors: [const Color(0xFFFFFF00), const Color(0xFF0099FF)],
      stops: [0.4, 1.0],
    );
    canvas.drawRect(
      rect,
      new Paint()..shader = gradient.createShader(rect),
    );
  }
  @override
  SemanticsBuilderCallback get semanticsBuilder {
    return (Size size) {
      // Annotate a rectangle containing the picture of the sun
      // with the label "Sun". When text to speech feature is enabled on the
      // device, a user will be able to locate the sun on this picture by
      // touch.
      var rect = Offset.zero & size;
      var width = size.shortestSide * 0.4;
      rect = const Alignment(0.8, -0.9).inscribe(new Size(width, width), rect);
      return [
        new CustomPainterSemantics(
          rect: rect,
          properties: new SemanticsProperties(
            label: 'Sun',
            textDirection: TextDirection.ltr,
          ),
        ),
      ];
    };
  }
  // Since this Sky painter has no fields, it always paints
  // the same thing and semantics information is the same.
  // Therefore we return false here. If we had fields (set
  // from the constructor) then we would return true if any
  // of them differed from the same fields on the oldDelegate.
  @override
  bool shouldRepaint(Sky oldDelegate) => false;
  @override
  bool shouldRebuildSemantics(Sky oldDelegate) => false;
}
```
See also:
 * [Canvas], the class that a custom painter uses to paint.
 * [CustomPaint], the widget that uses [CustomPainter], and whose sample
   code shows how to use the above `Sky` class.
 * [RadialGradient], whose sample code section shows a different take
   on the sample code above.
