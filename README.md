# 3D-9
## 粒子系统
> 参考 http://i-remember.fr/en 这类网站，使用粒子流编程控制制作一些效果， 如“粒子光环”

### 项目过程
> 先生成一个最普通的粒子光环,下面是粒子位置
```
public class particalPos{
	public float angle;
	public float radius;
	public float time;
	//public float speed;
	private float temp;
	private float xPos;
	private float yPos;

	public particalPos(float angle,float radius,float time){//四维变量
		this.angle = angle;
		this.radius = radius;
		this.time = time;


	}
	public float getX(){
		temp=angle / 180.0f * Mathf.PI;
		xPos = radius * Mathf.Cos(temp);
		return xPos;
	}
	public float getY(){
		temp=angle / 180.0f * Mathf.PI;
		yPos=radius * Mathf.Sin(temp);
		return yPos;
	}

}
```

```
public class  partical : MonoBehaviour {
	private ParticleSystem _ParticleSystem;
	private ParticleSystem.Particle[] _Particles;
	private particalPos[] _particalPos;

	public int count = 1000;
	public float size = 0.5f;
	//设置最大最小旋转半径
	public float minRadius = 25.0f;
	public float maxRadius = 30.0f;

	public float speed = 0.5f;
void Start () {
	_ParticleSystem = this.GetComponent<ParticleSystem>();
	_Particles = new ParticleSystem.Particle[count];

	_ParticleSystem.Emit (count);//发射
	_ParticleSystem.GetParticles (_Particles);//将particle连接到_ParticleSystem
	_particalPos = new particalPos[count];

	float angleTemp=0, radiusTemp = 0,radiusTemp1 = 0,radiusTemp2 = 0,timeTemp=0;
	for (int i = 0; i < count; i++) {
		angleTemp = Random.Range (0f,360f);
		_Particles[i].position=new Vector3 (_particalPos[i].getX(),_particalPos [i].getY(),0 );
	}
	_ParticleSystem.SetParticles (_Particles, count);
	}
```
```
  void Update () {
	for (int i = 0; i < count; i++) {
		if (i%7>2) {
			_particalPos[i].angle=((float)(_particalPos[i].angle+(0.3))%360;
		} else {
			_particalPos[i].angle=((float)(_particalPos[i].angle-(0.3))360;
		}
		_Particles [i].position = new Vector3 (_particalPos[i].getX(),_particalPos [i].getY(),0 );
	}
	_ParticleSystem.SetParticles (_Particles, count);
}
```
> 先生成最基本的旋转粒子系统，生成代码如上，生成的效果如下，但是发现有一个问题就是粒子系统在抖动。

>     gif动图连接（太大无法加载出来）  
http://imglf6.nosdn.127.net/img/Z281REhERnhNZlhBVVE4ejRLSGRHZ2pRQ3VGYTgwUEVzcUVkemV4cW0yTk5DMVRXOW9MM2RRPT0.gif  

> 调试发现是Particle system的simulation speed不为0导致粒子系统一直在抖动，于是修改对应的变量，修改方法如下：

```
    //修改simulationSpeed,避免震动
	var main=_ParticleSystem.main;
	main.simulationSpeed = 0f;
```

> 解决了抖动问题之后，加入反向旋转，以及为了呈现粒子内部旋转角速度比外部更快的效果，修改update里面的函数。效果如下：
```
for (int i = 0; i < count; i++) {
	if (i%7>2) {
		_particalPos[i].angle=((float)(_particalPos[i].angle+(0.3/(_particalPos[i].radius-minRadius/2)*minRadius)))%360;
	} else {
		_particalPos[i].angle=((float)(_particalPos[i].angle-(0.3/(_particalPos[i].radius-minRadius/2)*minRadius)))%360;
	}
	_Particles [i].position = new Vector3 (_particalPos[i].getX(),_particalPos [i].getY(),0 );
	}
```
  gif动图连接（太大无法加载出来）  
  http://imglf5.nosdn.127.net/img/Z281REhERnhNZlhBVVE4ejRLSGRHbERhYjZQTnZnb3hDRDdoVDg3cVhCemN2TU9KeHp1cWd3PT0.gif  
   
   
> 继续修改，为了使粒子的半径呈现随机移动的效果，加入PerlinNoise函数
```
  _particalPos[i].radius+= 0.06f * (Mathf.PerlinNoise(_particalPos[i].angle/360.0f, 0.0f))-0.03f;
```

> 又发现由于有的粒子可能会越跑越远，为了保证粒子始终在minRadius和maxRadius之间，添加下面的语句。
```
  //控制粒子位置
	_particalPos [i].radius = (_particalPos [i].radius) % (maxRadius-minRadius)+minRadius;
```
> 为了让粒子尽可能聚集在圆环中间，加入下面的语句:
```
  angleTemp = Random.Range (0f,360f);
	radiusTemp1 = Random.Range (minRadius, (maxRadius - minRadius) / 2 + minRadius);
	radiusTemp2 = Random.Range (maxRadius - minRadius) / 2 + minRadius, maxRadius);
	radiusTemp = Random.Range (radiusTemp1,radiusTemp2);
	_particalPos [i] = new particalPos (angleTemp, radiusTemp, timeTemp);
```
> 最终效果如下：(但是感觉效果还是不够，一方面是粒子的种类素材,一方面是受限于系统粒子数量太少)

  gif动图连接（太大无法加载出来）  
  
http://imglf6.nosdn.127.net/img/Z281REhERnhNZlhBVVE4ejRLSGRHaEErUWxSajBwYjJ6YWpUcU1vdXdEcGRMY0toamFkOHJ3PT0.gif


> 附上源代码
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;


public class particalPos{
	public float angle;
	public float radius;
	public float time;
	//public float speed;
	private float temp;
	private float xPos;
	private float yPos;

	public particalPos(float angle,float radius,float time){//四维变量
		this.angle = angle;
		this.radius = radius;
		this.time = time;


	}
	public float getX(){
		temp=angle / 180.0f * Mathf.PI;
		xPos = radius * Mathf.Cos(temp);
		return xPos;
	}
	public float getY(){
		temp=angle / 180.0f * Mathf.PI;
		yPos=radius * Mathf.Sin(temp);
		return yPos;
	}

}

public class  partical : MonoBehaviour {
	private ParticleSystem _ParticleSystem;
	private ParticleSystem.Particle[] _Particles;
	private particalPos[] _particalPos;

	public int count = 1000;
	public float size = 0.5f;
	//设置最大最小旋转半径
	public float minRadius = 25.0f;
	public float maxRadius = 30.0f;

	public float speed = 0.5f;



	void Start () {
		_ParticleSystem = this.GetComponent<ParticleSystem>();
		_Particles = new ParticleSystem.Particle[count];

		_ParticleSystem.Emit (count);//发射

		//修改simulationSpeed,避免震动
		var main=_ParticleSystem.main;
		main.simulationSpeed = 0f;

		_ParticleSystem.GetParticles (_Particles);//将particle连接到_ParticleSystem
		_particalPos = new particalPos[count];

		float angleTemp=0, radiusTemp = 0,radiusTemp1 = 0,radiusTemp2 = 0,timeTemp=0;
		for (int i = 0; i < count; i++) {
			angleTemp = Random.Range (0f,360f);

			radiusTemp1 = Random.Range (minRadius, (maxRadius - minRadius) / 2 + minRadius);
			radiusTemp2 = Random.Range ((maxRadius - minRadius) / 2 + minRadius, maxRadius);
			radiusTemp = Random.Range (radiusTemp1,radiusTemp2);



			_particalPos [i] = new particalPos (angleTemp, radiusTemp, timeTemp);
			_Particles[i].position=new Vector3 (_particalPos[i].getX(),_particalPos [i].getY(),0 );
		}

		_ParticleSystem.SetParticles (_Particles, count);
	}
	void Update () {

		for (int i = 0; i < count; i++) {
			if (i%7>2) {
				_particalPos[i].angle=((float)(_particalPos[i].angle+(0.3/(_particalPos[i].radius-minRadius/2)*minRadius)))%360;
			} else {
				_particalPos[i].angle=((float)(_particalPos[i].angle-(0.3/(_particalPos[i].radius-minRadius/2)*minRadius)))%360;
			}
			_particalPos[i].radius+= 0.06f * (Mathf.PerlinNoise(_particalPos[i].angle/360.0f, 0.0f))-0.03f;
			//控制粒子位置
			_particalPos [i].radius = (_particalPos [i].radius) % (maxRadius-minRadius)+minRadius;

			_Particles [i].position = new Vector3 (_particalPos[i].getX(),_particalPos [i].getY(),0 );
			//Debug.Log ("update: _particalPos["+i+"].getX()  "+_particalPos[i].getX());
		}
		//维持粒子数量不变
		/*if(_ParticleSystem.particleCount<count){
			_ParticleSystem.Emit (count - _ParticleSystem.particleCount);
		}*/
		//Debug.Log ("_ParticleSystem.particleCount: " + _ParticleSystem.particleCount);

		_ParticleSystem.SetParticles (_Particles, count);
	}
}
```
