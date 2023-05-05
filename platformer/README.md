# Ember Quest

simple game from [flame engine](https://github.com/flame-engine/flame) tutorials.<p>
<img src=screen.png />

## GameWidget.controlled() 생성자
* gameFactory: EmberQuestGame.new
  * EmberQuestGame.new: .new는 대표 생성자 함수를 참조한다.
* overlayBuilderMap으로 overlay를 생성한다.
  * FlameGame.overlays로 게임 위에 보이는 overlay widget을 관리한다.

## EmberQuestGame(extends FlameGame with HasCollisionDetection, HasKeyboardHandlerComponents)
* HasCollisionDetection mixin for supporting collision detection
* HasKeyboardHandlerComponents mixin for supporting keyboard
* @override onLoad()에서 게임을 초기화 함(initializeGame 함수)
  * 게임 객체 ground, platform, star, water enemy를 생성한다. 
    * segment = 10 x 10 blocks(block size = 64 x 64)으로 나눈다.
    * 화면 크기보다 더 많은 개수의 segment를 생성한다.
    * 화면이 스크롤 되면, 추가로 segment를 생성한다.
    * 지나간 segment는 제거한다.
  * EmberPlayer 객체를 추가한다.
  * Hud 객체를 추가한다.

## overlay widgets - MainMenu, GameOver class
* MainMenu
  * Play 버튼을 누르면 overlay가 사라진다.
* GameOver
  * Play Again 버튼을 누르면,
    * game을 재시작하고,
    * overlay가 사라진다.

## EmberPlayer(extends SpriteAnimationComponent with KeyboardHandler, CollisionCallbacks, HasGameRef<EmberQuestGame>) class
* SpriteAnimationComponent
  * onLoad() 함수에서 SpriteAnimation을 생성한다.
    * SpriteAnimation.fromFrameData()로 생성하며,
    * SpriteAnimationData.sequenced() 함수로 animation data를 생성한다.
      * 하나의 이미지에 여러 개의 texture로 구성하고,
      * 순차적으로 반복되는 animation frame을 생성한다.
      * SpriteAnimationData.variable() 또는 range() 함수를 사용할 수 있다.
    * CircleHitbox()를 child component로 추가한다.
      * RectangleHitbox() 또는 PolygonHitbox()를 사용할 수 있다.
* CollisionCallbacks
  * @override onCollision(other)
    * other is GroundBlock || other is PlatformBlock
      * 아래 방향으로 충돌된 경우, isOnGround를 true로 설정한다.
    * other is Star
      * other.removeFromParent() ==> Star를 제거한다.
      * game.starsCollected를 증가시킨다. ==> Hud.update() 함수에서 update한다.
        * Component.update() 함수는 주기적으로 실행된다.
    * other is WaterEnemy
      * game.health를 감소시킨다. ==> HeartHealthComponent.update() 함수에서 update한다.
        * available / unavailable sprite를 상태에 따라서 유지한다.
      * OpacityEffect.fadeOut()를 추가해서 player를 투명하도록 한다.
* KeyboardHandler
  * @override onKeyEvent(keysPressed)
    * horizontalDirection(-1, 0, 1), hasJumped(true or false)를 결정한다.
* @override update()
  * player의 위치를 이동시킨다. jump가 구현되어 있다.
  * 좌/우 방향으로 이동할 때, sprite를 바꾼다.
    * PositionComponent.flipHorizontally() 함수
    * flipVertically(), flipVerticallyAroundCenter() 등이 있다.
  * player가 x축의 중앙을 넘어가면 x를 증가시키는 대신에 게임 전체를 왼쪽으로 옮긴다.
    * game.objectSpeed를 -moveSpeed로 설정한다.
    * ground, platform, star, water enemy의 update() 함수에서 game.objectSpeed를 적용한다.
    * 전체적으로 움직이는 것이므로 CameraComponent를 사용하는 것이 좋을 것 같다.

## Collisions
* [Flame Engine의 충돌 검출에 대한 설명 참고](https://docs.flame-engine.org/latest/flame/collision_detection.html)
  * 충돌 검출을 사용하려면 Game을 선언할 때, HasCollisionDetection mixin을 사용한다.
    * class EmberQuestGame extends FlameGame with HasCollisionDetection
  * 또는 World 수준에서 충돌 검출을 사용할 수도 있다.
    * class CollisionDetectionWorld extends World with HasCollisionDetection {}
  * 그리고는, ShapeHitbox를 child component로 추가하면, 자동으로 충돌 검출이 동작한다.
  * CollisionCallbacks mixin으로 충돌에 대한 callback을 구현한다.
* 각각 ShapeHitbox는 CollisionType을 가진다.
  * CollisionType.active: active/passive hitbox와 충돌한다.
  * CollisionType.passive: active hitbox와 충돌한다.
  * CollisionType.inactive: 충돌하지 않는다.
* A broad phase is the first step of collision detection where potential collisions are calculated.
  * Quad Tree broad phase can be used with HasQuadTreeCollisionDetection mixin.
* [Forge2D](https://github.com/flame-engine/forge2d) recommended for the better physics engine
  * Forge2D is a Dart port of the Box2D physics engine.
  * [flame_forge2d](https://github.com/flame-engine/flame_forge2d) is the bridge between Flame and Forge2D

## HeartHealthComponent(extends SpriteGroupComponent<HeartState>) class
* SpriteGroupComponent<T> class
  * A PositionComponent that can have multiple Sprites and render the one mapped with the current key.
  * enum HeartState { available, unavailable, }
  * Map<T, Sprite>? sprites <-- 상태(key)에 대한 sprite map  
  * T? current <-- 현재 상태(key)