## 点击并移动 官方教程 代码解析

官方教程链接 [Point and click movement](https://developer.playcanvas.com/en/tutorials/point-and-click-movement/)

脚本实现的预定目标：通过相机点击地面，角色移动到地面的对应位置

```javascript
PointAndClick.attributes.add('cameraEntity', {type: 'entity', title: 'Camera Entity'});
PointAndClick.attributes.add('playerEntity', {type: 'entity', title: 'Player Entity'});
PointAndClick.attributes.add('playerSpeed', {type: 'number', default: 0, title: 'Player Speed'});
```

*暴露变量到面板*，指定主要相机和主要玩家的实体，生命玩家移动的速度

```javascript
this.groundShape = new pc.BoundingBox(new pc.Vec3(0, 0, 0), new pc.Vec3(4, 0.001, 4));
```

使用`boundingbox` API*生成碰撞箱*。第一个参数是center，表示碰撞箱的中心点第二个参数是halfExtents，表示在各个轴上的半径。

```javascript
this.direction = new pc.Vec3();
this.distanceToTravel = 0;
this.targetPosition = new pc.Vec3();
```

> 这里要注意一点，也就是变量的作用域：
>
> 使用`var variablename；`的作用域是函数，而使用`this.varaiablename`的，作用域则是整个脚本。



好的，这里，我们先来构思一下：

我们想得到一个玩家到目的地的**方向**，移动的**距离**。我们的目标点需要通过**相机**的**位置**和**角度**确定，所以我们首先从**相机的坐标**出发，经过**单击事件的世界坐标**，创建一个射线，找到射线和地面的**交点**。然后计算**玩家到交点**的**方向和距离**。

针对移动的过程，官方提供的解决方案是在update方法中不断移动entity。我一开始的想法是递归调用，现在想想这种递归调用可能会有风险。

所以在这里我们声明了三个很重要的变量，这两个变量会在update方法中持续被更新和调用。

`direction`是是角色移动的方向

`distanceToTravel`是角色移动的距离。

`targetposition`是目标坐标。

*注册鼠标点击和触摸开始事件*

```javascript
this.app.mouse.on(pc.EVENT_MOUSEDOWN, this.onMouseDown, this);

    if (this.app.touch) {
        this.app.touch.on(pc.EVENT_TOUCHSTART, this.onTouchStart, this);
    }
```

我们先跳到onTouchStart这个方法：

```javascript
if (event.touches.length == 1) {
        this.doRayCast(event.touches[0]);
        event.event.preventDefault();
```

首先我们来看TouchEvent这个api，这是一个输入事件Input Event，`event.touches`是一个数组，包含这多点触控中的多个单击，`touch[n]`中包含`screenX`和`screenY`两个属性。

`TOUCHSTART`则是输入事件Input Event的一个事件。除去这个之外，还有其他的几个[事件](https://developer.playcanvas.com/en/user-manual/user-interface/input/#input-events)。

`event.event.preventDefault();`则是用来取消Chrome在移动端对鼠标单击事件的模拟。[详见官方手册](https://developer.playcanvas.com/en/user-manual/user-interface/input/#mouse-and-touch-event-conflict-on-google-chrome)



`touch`事件和`mouse`事件都会将屏幕坐标以`screenPosition`的变量名传进`doRayCast`函数

在这个函数之前，有两个变量声明

```javascript
PointAndClick.ray = new pc.Ray();
PointAndClick.hitPosition = new pc.Vec3();
```

注意第一个变量，它是[Ray类型](https://developer.playcanvas.com/en/api/pc.Ray.html)。它包含两个属性direction和origin。表示射线的起点和终点，这两个属性都是Vec3类。origin是射线的起点，direction是射线的三维方向。

> 要记住，`Vec3`是一个向量，它的（x,y,z）值既可以指一个方向，也可以指一个坐标

接下来我们用到了`BoudingBox.intersectsRay(ray, hitPosition)`这个方法，这个方法，传入一个`ray`类型，并将`ray`和`boundingbox`的交汇点写到`hitPosition`里，并返回一个**布朗值**

如果存在交汇点，则返回的`result`为`true`，那么就会将`hitPosition`传进`movePlayerTo`函数里。



进入movePlayerTo函数中，首先将worldPosition复制进targetPosition里，然后将targetPosition设为角色的平视y轴值。这样的话，角色的移动距离才能正确地计算。

```javascript
this.targetPosition.y = this.playerEntity.getPosition().y;
```

这个函数的作用主要是**计算移动方向和移动距离**，也就是说，`targetPosition`复制`worldPosition`的意义在于，可以在`update`方法中调用这个变量，而update才是用于移动角色的函数。

```javascript
this.distanceToTravel = this.direction.length();
```

使用方向和长度的向量计算出距离，然后将这个向量标准化。

```javascript
if (this.distanceToTravel > 0) {
        // 转换为单位方向向量
        this.direction.normalize();

        this.playerEntity.lookAt(this.targetPosition);
    } else {
        this.direction.set(0, 0, 0);
```

判断是否需要移动，`distanceToTravel`，**作为一个保险机制**，如果`direction`得长度为0，则将`direction`重置为`ZERO`。

> 我之前写法线的东西，每个地方Normalize一次，一个数据能标准化10次。——素质

最后使用`entity.lookAt`方法，将实体转向指向的方向。

最后我们会回到`update`方法：

```javascript
if (this.direction.lengthSq() > 0)
```

这里再次检查一遍`direction`是否存在存在长度。这里的检查是因为，`update`从`initialize`开始就开始执行，可能会发生不可预知的错误。在这里，如果去掉这个判断，会在`update`中将实体移动到`（0,0,0）`的位置上。

```javascript
var d = this.playerSpeed * dt;
var newPosition = PointAndClick.newPosition;
       
newPosition.copy(this.direction).scale(d);
newPosition.add(this.playerEntity.getPosition());        
this.playerEntity.setPosition(newPosition);     
```

由于`direction`现在是一个方向向量，我们可以用这个方向向量乘以速度获得在**方向上移动的距离，并将这个距离加到实体坐标上，得到世界坐标**。



在`this.distanceToTravel -= d;`不断将`distanceToTravel`减去之后，一旦此变量降为0，则停止运动，将`direction`重置为`ZERO`。

-----------------------



代码全文

```javascript
var PointAndClick = pc.createScript('pointAndClick');

PointAndClick.attributes.add('cameraEntity', {type: 'entity', title: 'Camera Entity'});
PointAndClick.attributes.add('playerEntity', {type: 'entity', title: 'Player Entity'});
PointAndClick.attributes.add('playerSpeed', {type: 'number', default: 0, title: 'Player Speed'});

// initialize code called once per entity
PointAndClick.prototype.initialize = function() {
    this.groundShape = new pc.BoundingBox(new pc.Vec3(0, 0, 0), new pc.Vec3(4, 0.001, 4));
    this.direction = new pc.Vec3();
    this.distanceToTravel = 0;
    this.targetPosition = new pc.Vec3();
    
    // Register the mouse down and touch start event so we know when the user has clicked
    this.app.mouse.on(pc.EVENT_MOUSEDOWN, this.onMouseDown, this);

    if (this.app.touch) {
        this.app.touch.on(pc.EVENT_TOUCHSTART, this.onTouchStart, this);
    }
};


PointAndClick.newPosition = new pc.Vec3();

// update code called every frame
PointAndClick.prototype.update = function(dt) {
    if (this.direction.lengthSq() > 0) {
        // Move in the direction at a set speed
        var d = this.playerSpeed * dt;
        var newPosition = PointAndClick.newPosition;
       
        newPosition.copy(this.direction).scale(d);
        newPosition.add(this.playerEntity.getPosition());        
        this.playerEntity.setPosition(newPosition);     
        
        this.distanceToTravel -= d;
        
        // If we have reached our destination, clamp the position 
        // and reset the direction
        if (this.distanceToTravel <= 0) {
            this.playerEntity.setPosition(this.targetPosition);
            this.direction.set(0, 0, 0);
        }
    }
};


PointAndClick.prototype.movePlayerTo = function (worldPosition) {
    this.targetPosition.copy(worldPosition);
        
    // Assuming we are travelling on a flat, horizontal surface, we make the Y the same
    // as the player
    this.targetPosition.y = this.playerEntity.getPosition().y;

    this.distanceTravelled = 0;
    
    // 用ray碰撞地面得出的地面坐标，减去玩家的世界坐标，得到一个向量，是从玩家出发到目的地的向量
    this.direction.sub2(this.targetPosition, this.playerEntity.getPosition());
    
    // Get the distance the player needs to travel for
    this.distanceToTravel = this.direction.length();
    
    if (this.distanceToTravel > 0) {
        // 将原本既包含方向数据也包含距离的向量转换为单位向量
        this.direction.normalize();

        this.playerEntity.lookAt(this.targetPosition);
    } else {
        this.direction.set(0, 0, 0);
    }
};


PointAndClick.prototype.onMouseDown = function(event) {
    if (event.button == pc.MOUSEBUTTON_LEFT) {
        this.doRayCast(event);
    }
};


PointAndClick.prototype.onTouchStart = function (event) {
    // On perform the raycast logic if the user has one finger on the screen
    if (event.touches.length == 1) {
        this.doRayCast(event.touches[0]);
        event.event.preventDefault();
    }    
};


PointAndClick.ray = new pc.Ray();
PointAndClick.hitPosition = new pc.Vec3();

PointAndClick.prototype.doRayCast = function (screenPosition) {
    // Initialise the ray and work out the direction of the ray from the a screen position
    var ray = PointAndClick.ray;
    var hitPosition = PointAndClick.hitPosition;

    //使用点击的空间坐标减去相机的的location，得到一个方向向量
    this.cameraEntity.camera.screenToWorld(screenPosition.x, screenPosition.y, this.cameraEntity.camera.farClip, ray.direction); 
    ray.origin.copy(this.cameraEntity.getPosition());
    ray.direction.sub(ray.origin).normalize();
    
    // Test the ray against the ground
    var result = this.groundShape.intersectsRay(ray, hitPosition);  
    if (result) {
        this.movePlayerTo(hitPosition);
    }  
};
```

