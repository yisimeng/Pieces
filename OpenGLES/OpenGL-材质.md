# 材质

不同物体对光的反应不同。通过环境光照、漫反射光照、镜面光照和**反光度**来定义材质的颜色。

```
// 材质
struct Material {
	vec3 ambient; // 环境光照
	vec3 diffuse; // 漫反射
	vec3 specular;// 镜面光照
	float shininess; //反光度
};
uniform Material material; //通过uniform 设置值

// 光照属性
struct Light {
	vec3 position; // 位置
	vec3 ambient;  // 环境
	vec3 diffuse;  // 漫反射
	vec3 specular; // 镜面
};
uniform Light light; // 通过uniform设置值
```