# space_shooter

simple game from [flame engine](https://github.com/flame-engine/flame) tutorials.
* SpriteComponent를 확장해서 Player 클래스를 선언한다.
* Player 객체를 생성해서 FlameGame에 추가한다 - add(player).
* PanDetector로 DRAG 입력을 받아서, player의 position 값을 수정한다.
  * SpriteComponent의 position 값을 수정하기만 하면, 위치가 변경됨을 알 수 있다.

## Flame 기본 클래스
* GameWidget: FlameGame을 보이는 Flutter Widget(Stateful) 
  * game : FlameGame 객체
* FlameGame: Flame Game의 root component (Component를 상속함)
* Component: Flame Game의 기본적인 building block.
  * ComponentSet children으로 자식 component를 가짐.
  * FlameGame이 game tree의 root node임.
* SpriteComponent: Sprite를 위치, 크기로 렌더링하는 Component.
* Sprite: Image의 일부
* PanDetector: mixin to receive pan events.