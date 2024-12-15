### Documentation

For this project i have created five elements with babylonJS and typescript, the 5th and final element is a display of all the previous work with the ability to switch between each of the scenes.

There are three scenes, each demonstrating different things that can be done with babylonJS:

## Scene 1

The first scene demonstrates the use of basic shapes, lighting, shadows and motion. It is the simplest of the three and has only two important files.

createStartScene.ts is the main file containing all the functions, it is split into 3 parts:

1. All required modules are imported.

```typescript
import {
  Scene,
  ArcRotateCamera,
  Vector3,
  Vector4,
  HemisphericLight,
  SpotLight,
  MeshBuilder,
  Mesh,
  Light,
  Camera,
  Engine,
  StandardMaterial,
  Texture,
  Color3,
  Space,
  ShadowGenerator,
  PointLight,
  DirectionalLight,
} from '@babylonjs/core';
```

2. A function section where all the objects that will appear in the scene are defined.

```typescript
function createBox(
  scene: Scene,
  px: number,
  py: number,
  pz: number,
  sx: number,
  sy: number,
  sz: number,
  rotation: boolean
) {
  let box = MeshBuilder.CreateBox('box', { size: 1 }, scene);
  box.position = new Vector3(px, py, pz);
  box.scaling = new Vector3(sx, sy, sz);
  box.receiveShadows = true;

  if (rotation) {
    scene.registerAfterRender(function () {
      box.rotate(new Vector3(10, 7, 5) /*axis*/, 0.4 /*angle*/, Space.LOCAL);
    });
  }
  return box;
}
```

3. the export section where these objects will be added to the scene. the values such as size and position can either be defined within the function or in the export as shown here.

```typescript
export default function createStartScene(engine: Engine) {
  interface SceneData {
    scene: Scene;
    box?: Mesh;
    faceBox?: Mesh;
    light?: Light;
    hemisphericLight?: HemisphericLight;
    directionalLight?: DirectionalLight;
    sphere?: Mesh;
    torus?: Mesh;
    ground?: Mesh;
    camera?: Camera;
  }

  let that: SceneData = { scene: new Scene(engine) };
  that.scene.debugLayer.show();

  that.hemisphericLight = createHemiLight(that.scene);
  that.directionalLight = createDirectionalLight(that.scene);
  that.faceBox = createFacedBox(that.scene, -10, 2.5, 0, 0.5, 1, 1);
  that.light = createAnyLight(
    that.scene,
    2,
    -10,
    5,
    0,
    0.75,
    0.12,
    0.91,
    that.faceBox
  );

  that.box = createBox(that.scene, -13, 2.5, 0, 0.5, 5, 9, false);
  that.box = createBox(that.scene, -8, 2.5, 0, 0.5, 9, 9, false);
  that.box = createBox(that.scene, -3, 2.5, 0, 0.5, 13, 9, false);
  that.box = createBox(that.scene, 2, 2.5, 0, 0.5, 16, 9, false);
  that.box = createBox(that.scene, 7, 2.5, 0, 0.5, 20, 9, false);
  that.box = createBox(that.scene, 12, 2.5, 0, 0.5, 24, 9, false);

  that.camera = createArcRotateCamera(that.scene);
  return that;
}
```

this is the file structure that is used in all babylonJS files like this. the createBox function is called multiple times, each with a different position and size to make the scaling blocks that appear in scene 1.

## Scene 2

Scene 2 was made to demonstrate a 3D environmnet featuring a heightmap and a skybox.

```typescript
function createTerrain(scene: Scene) {
  const largeGroundMat = new StandardMaterial('largeGroundMat');
  largeGroundMat.diffuseTexture = new Texture(
    './assets/environments/valleygrass.png'
  );

  const largeGround = MeshBuilder.CreateGroundFromHeightMap(
    'largeGround',
    './assets/environments/Height.png',
    {
      width: 300,
      height: 300,
      subdivisions: 100,
      minHeight: 0,
      maxHeight: 40,
    },
    scene
  );
  largeGround.material = largeGroundMat;
  largeGround.position.y = -20;
}
```

this is the function that is used to create the terrain. a bitmap terrain is made from a heightmap which is converted into a mesh and a texture which is used with it. the size, height and number of polygons are also defined here.

```typescript
function createSky(scene: Scene) {
  const skybox = MeshBuilder.CreateBox('skyBox', { size: 150 }, scene);
  const skyboxMaterial = new StandardMaterial('skyBox', scene);
  skyboxMaterial.backFaceCulling = false;
  skyboxMaterial.reflectionTexture = new CubeTexture(
    './assets/textures/skybox/skybox',
    scene
  );
  skyboxMaterial.reflectionTexture.coordinatesMode = Texture.SKYBOX_MODE;
  skyboxMaterial.diffuseColor = new Color3(0, 0, 0);
  skyboxMaterial.specularColor = new Color3(0, 0, 0);
  skybox.material = skyboxMaterial;
  return skybox;
}
```

This is the function that is used to create the skybox. The skybox is made of 6 images which are combined together and then displayed in a 360 view within the scene.

## Scene 3

Scene 3 was created to demonstrate a playable character and interactive physics

```typescript
function importMeshA(scene: Scene, x: number, y: number) {
  let item: Promise<void | ISceneLoaderAsyncResult> =
    SceneLoader.ImportMeshAsync(
      '',
      './assets/models/',
      'dummy3.babylon',
      scene
    );

  item.then((result) => {
    let character: AbstractMesh = result!.meshes[0];
    character.position.x = x;
    character.position.y = y + 0.1;
    character.scaling = new Vector3(1, 1, 1);
    character.rotation = new Vector3(0, 1.5, 0);
  });
  return item;
}
```

first a new mesh is created that the player will get to control, this is the blue mannequin that appears in the scene.

```typescript
function createBox1(scene: Scene) {
  let box1 = MeshBuilder.CreateBox('box', { width: 1, height: 1 }, scene);
  box1.position.x = -1;
  box1.position.y = 4;
  box1.position.z = 1;
  return box1;
}
```

secondly a new box function will be used to demonstrate physics.

the third scene involves two new files, collisionDeclaration.ts and keyActionManager.ts

```typescript
const groundAggregate = new PhysicsAggregate(
  runScene.ground,
  PhysicsShapeType.BOX,
  { mass: 0, restitution: 0.2, friction: 0.7 },
  runScene.scene
);
groundAggregate.body.setCollisionCallbackEnabled(true);

const boxAggregate = new PhysicsAggregate(
  runScene.box1,
  PhysicsShapeType.BOX,
  { mass: 1, restitution: 0.3, friction: 0.7 },
  runScene.scene
);
boxAggregate.body.setCollisionCallbackEnabled(true);

runScene.player!.then((result: void | ISceneLoaderAsyncResult) => {
  let character: AbstractMesh = result!.meshes[0];
  character.rotation = new Vector3(0, 0.5, 0);

  const playerAggregate = new PhysicsAggregate(
    character,
    PhysicsShapeType.CAPSULE,
    { mass: 0.1, restitution: 1, friction: 1 },
    runScene.scene
  );
  playerAggregate.body.setMassProperties({
    inertia: new Vector3(0, 0.0, 0.0),
  });
  playerAggregate.body.setAngularVelocity(new Vector3(0, 12, 0));

  playerAggregate.body.applyImpulse(new Vector3(0, 0, 0), character.position);

  playerAggregate.body.disablePreStep = false;
  playerAggregate.body.setCollisionCallbackEnabled(true);
});
```

the collisionDeclaration.ts file introduces Havok physics into the scene, here the player, the ground the player will walk on and the box the player will move are referenced.

there are multiple files introduced to handle controls and animations for the player.

```typescript
      export function keyActionManager(scene: Scene) {
    scene.actionManager.registerAction(
        new ExecuteCodeAction(
          {
            trigger: ActionManager.OnKeyDownTrigger,
          },
          function (evt) {
            if (keyDown == 0){keyDown ++}
            keyDownMap[evt.sourceEvent.key] = true;
          }
        )
```

keyActionManager.ts is introduced to detect keypresses.

```typescript
      runScene.player.then((result) => {
      let characterMoving: Boolean = false;
      let character: AbstractMesh = result!.meshes[0];
      if (keyDownMap["w"] || keyDownMap["ArrowUp"]) {
        character.position.x -= 0.1;
        character.rotation.y = (3 * Math.PI) / 2;
        characterMoving = true;
      }
```

createRunScene.ts features code that moves the player based on keys pressed.

```typescript
export function bakedAnimations(myscene: Scene, skeleton: Skeleton) {
  // use baked in animations
  animScene = myscene;
  animSkeleton = skeleton;
  skeleton.animationPropertiesOverride = new AnimationPropertiesOverride();
  skeleton.animationPropertiesOverride.enableBlending = true;
  skeleton.animationPropertiesOverride.blendingSpeed = 0.05;
  skeleton.animationPropertiesOverride.loopMode = 1;

  walkRange = skeleton.getAnimationRange('YBot_Walk');
  runRange = skeleton.getAnimationRange('YBot_Run');
  leftRange = skeleton.getAnimationRange('YBot_LeftStrafeWalk');
  rightRange = skeleton.getAnimationRange('YBot_RightStrafeWalk');
  idleRange = skeleton.getAnimationRange('YBot_Idle');
  console.log(idleRange);
}
```

bakedAnimations.ts extracts animations that are baked into the player model itself so they can be used and then exports them to createRunScene.ts to be played alongside keypresses.

Element 5 features a GUI that allows the player to switch between three scenes, this was done by featuring buttons on each scene and changing a value within index.ts to change the scene when a button was pressed.

```typescript
scenes[0] = element1(eng);
scenes[1] = element2(eng);
scenes[2] = element4(eng);

scene = scenes[0].scene;
setSceneIndex(0);

export default function setSceneIndex(i: number) {
  eng.runRenderLoop(() => {
    scenes[i].scene.render();
  });
}
```

Within element 5, each scene has its own file and these are defined as scenes. the renderer is set to display one of these scenes depending on which number setSceneIndex is currently set.

```typescript
function createSceneButton(
  scene: Scene,
  name: string,
  index: string,
  x: string,
  y: string,
  advtex: { addControl: (arg0: Button) => void },
  Sindex: number
) {
  var button: Button = Button.CreateSimpleButton(name, index);
  button.left = x;
  button.top = y;
  button.width = '180px';
  button.height = '35px';
  button.color = 'white';
  button.cornerRadius = 20;
  button.background = 'green';
  const buttonClick: Sound = new Sound(
    'MenuClickSFX',
    './assets/audio/menu-click.wav',
    scene,
    null,
    {
      loop: false,
      autoplay: false,
    }
  );
  button.onPointerUpObservable.add(function () {
    buttonClick.play();
    setSceneIndex(Sindex);
  });
  advtex.addControl(button);
  return button;
}
```

within each scene file the GUI is defined and linked to the setSceneIndex in the index.ts file, so that clicking them will change that value and transition to another scene.

```typescript
let button1: Button = createSceneButton(
    scene,
    "but1",
    "Element 1",
    "-700px",
    "0px",
    advancedTexture,
    0
  );
  let button2: Button = createSceneButton(
    scene,
    "but2",
    "Element 2",
    "-700px",
    "-60px",
    advancedTexture,
    1
  );
  let button3: Button = createSceneButton(
    scene,
    "but3",
    "Element 3",
    "-700px",
    "-120px",
    advancedTexture,
    2
```

the final value within each button is the setSceneIndex value.

## Assets used

"Landscape" (https://skfb.ly/WFuR) by mhart is licensed under Creative Commons Attribution (http://creativecommons.org/licenses/by/4.0/).
