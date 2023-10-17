2D物理引擎 Box2D for javascript Games -- 番外篇

此篇，并不是书中的篇符，而是通过希望通过结合实际的 canvas 绘图库实现 box2d 物理引擎在各绘图库上应用，绘图库网上有很多现成的

如：createjs, pixi.js 等，Egret或者其它游戏引擎有自己的物理引擎扩展库，所以就不说了。

现在通过之前的学习，基本掌握了刚体等基础概念。那如何如何应用于现实画面中呢？

Box2d 只是模拟了物体，是虚拟的，如果不是通过 debug 是看不到任何画面的，要让用户看到画面，必须得结合 canvas 绘图能力

自己直接操作 canvas 绘图的原始 API 太麻烦，所以就有了 createjs、pixi.js、fabric.js 等其它流行的 canvas 库.

以下都以 createJS 代替 canvas，当然你用其它库或者直接操作 canvas 也都可以

 

先上效果图

![image](https://img2018.cnblogs.com/blog/405426/201902/405426-20190217151129471-1564337733.png)
 

完成代码位于

https://github.com/willian12345/Box2D-for-Javascript-Games/blob/master/extra.html

 

Box2d 呈现于 createJS，贴上图的基本原理，就是将物理引擎世界中刚体的所有状态复制到 createJS 舞台对象！

 
```
function init() {
         var   b2Vec2 = Box2D.Common.Math.b2Vec2
            ,  b2AABB = Box2D.Collision.b2AABB
            ,  b2BodyDef = Box2D.Dynamics.b2BodyDef
            ,  b2Body = Box2D.Dynamics.b2Body
            ,  b2FixtureDef = Box2D.Dynamics.b2FixtureDef
            ,  b2Fixture = Box2D.Dynamics.b2Fixture
            ,  b2World = Box2D.Dynamics.b2World
            ,  b2MassData = Box2D.Collision.Shapes.b2MassData
            ,  b2PolygonShape = Box2D.Collision.Shapes.b2PolygonShape
            ,  b2CircleShape = Box2D.Collision.Shapes.b2CircleShape
            ,  b2DebugDraw = Box2D.Dynamics.b2DebugDraw
            ,  b2MouseJointDef =  Box2D.Dynamics.Joints.b2MouseJointDef
            ;
         var worldScale = 30; // box2d中以米为单位，1米=30像素
         var gravity = new b2Vec2(0, 5);
         var sleep = true;
         var world;
         var stage,debug;


         function main(){
            stage = new createjs.Stage("canvas");
            debug = new createjs.Stage("debug");

            setupPhysics();

            debugDraw();

            debug.on("stagemousedown", stagemousedown);

            createjs.Ticker.timingMode = createjs.Ticker.RAF;
            createjs.Ticker.on("tick", function(){
               stage.update();
               world.DrawDebugData(); // 为了显示出createjs对象，这里不再绘制box2d对象至canvas
               world.Step(1/30, 10, 10);// 更新世界模拟
               world.ClearForces(); // 清除作用力
            });
         }
         main();


         function Ball(){
            this.view = new createjs.Bitmap('soccer.png');
            this.view.regX = this.view.regY = 50;

            // 创建box2d球形体
            var bodyDef = new b2BodyDef();
            bodyDef.position.Set(Math.random()*640 / worldScale, 0/worldScale);
            bodyDef.type = b2Body.b2_dynamicBody
            bodyDef.userData = 0;
            var circleShape = new b2CircleShape(50 / worldScale);
            var fixtureDef = new b2FixtureDef();
            fixtureDef.shape = circleShape;
            fixtureDef.density = 1;
            fixtureDef.restitution = .4
            fixtureDef.friction = .5;
            this.view.body = world.CreateBody(bodyDef);
            this.view.body.CreateFixture(fixtureDef);

            this.view.on("tick", function(){
               // 让createjs的bitmap对象实时复制box2d对象的位置与旋转角度
               this.x = this.body.GetPosition().x * worldScale;
               this.y = this.body.GetPosition().y * worldScale;
               this.rotation = this.body.GetAngle() * (180 / Math.PI);
            });
         }
         
         function setupPhysics(){
            world = new b2World(new b2Vec2(0, 50), true);
            floor();  
         }

         function stagemousedown(){
            var b = new Ball();
            stage.addChild(b.view); // 将产生的createjs对象添加至舞台上
         }

         function floor(){
            var bodyDef = new b2BodyDef();
            bodyDef.position.Set(320/worldScale, 465/worldScale);
            var polygonShape = new b2PolygonShape();
            polygonShape.SetAsBox(320/worldScale, 15/worldScale);
            var fixtureDef = new b2FixtureDef();
            fixtureDef.shape = polygonShape;
            fixtureDef.restitution = .4;
            fixtureDef.friction = .5;
            var theFloor = world.CreateBody(bodyDef);
            theFloor.CreateFixture(fixtureDef);
         }

         //setup debug draw
         function debugDraw(){
            var debugDraw = new b2DebugDraw();
            debugDraw.SetSprite(debug.canvas.getContext('2d'));
            debugDraw.SetDrawScale(worldScale);
            debugDraw.SetFillAlpha(0.5);
            debugDraw.SetFlags(b2DebugDraw.e_shapeBit | b2DebugDraw.e_jointBit);
            world.SetDebugDraw(debugDraw);
         }
      };
```

注意：**分别使用两个 canvas**

一个用于渲染的 dom id 为 "canvas" 对象

```
new createjs.Stage("canvas");
```

另一个用于 Box2d  debug 渲染  dom id 为 "debug"

```
debug = new createjs.Stage("debug");
```

这一句引入一张图片

```
this.view = new createjs.Bitmap('soccer.png');
```

通过 createjs 的 Bitmap 对象读取图片，创建一个足球

this.view 这个显示对象即 createjs 的 Bitmap 对象，用于显示在舞台即 canvas 上

```
this.view.on("tick", function(){
    // 让createjs的bitmap对象实时复制box2d对象的位置与旋转角度
    this.x = this.body.GetPosition().x * worldScale;
    this.y = this.body.GetPosition().y * worldScale;
    this.rotation = this.body.GetAngle() * (180 / Math.PI);
});
```

在 Bitmap 对象上侦听 tick 事件，tick 事件回调，每一帧调用一次

在每帧中将刚体的 x,y 位置属性与角度属性复制到 createJS 的显示对象上，就完成了结合

 
注释掉这一句，就可以隐藏掉 Box2Djs 的调试状态变成一个正常的带物理效果的足球了

// debugDraw();

 

更多关于createJS请至官网或者搜索相关知识，你也完成可以用其它绘图库完成一样的操作

相关系列：

HTML5 之 2D物理引擎 Box2D for javascript Games 系列 第一章

 

----
注：转载请注明出处博客园：王二狗 Sheldon 池中物 (willian12345@126.com)

https://github.com/willian12345