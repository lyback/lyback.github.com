---
layout:     post
title:      "Unity-UGUI-Image扩展-圆形和方形裁剪"
subtitle:   "Image_Extension_Shape"
date:       2019-04-10 09:00:00
author:     "ly"
header-img: "img/post-bg-Unity.jpg"
tags:
    - UGUI
---

# **Image源码分析**
在对Image进行扩展之前，我们首先要了解Image源代码的一些原理，后面我们将继承Image类来进行扩展它的功能，[UGUI源码链接](https://bitbucket.org/Unity-Technologies/ui/src/0651862509331da4e85f519de88c99d0529493a5/?at=2018.3%2Fstaging)  

另外推荐阅读这篇文章[参考链接](https://aillieo.cn/post/2018-07-20-unity-3d-ugui-source-code-10/)，是我看到比较好的分析UGUI源码的文章，这里我只简单挑一些扩展需要用到的关键方法进行说明。

首先我们知道Image有四种绘制类型，在Image类一开始就定义了：  
```
public enum Type
{
    Simple,
    Sliced,
    Tiled,
    Filled
}
```
然后在Image类中的OnPopulateMesh()函数中会根据不同类型调用不同绘制方法：
```
protected override void OnPopulateMesh(VertexHelper toFill)
{
    if (activeSprite == null)
    {
        base.OnPopulateMesh(toFill);
        return;
    }
    switch (type)
    {
        case Type.Simple:
            GenerateSimpleSprite(toFill, m_PreserveAspect);
            break;
        case Type.Sliced:
            GenerateSlicedSprite(toFill);
            break;
        case Type.Tiled:
            GenerateTiledSprite(toFill);
            break;
        case Type.Filled:
            GenerateFilledSprite(toFill, m_PreserveAspect);
            break;
    }
}
```
接着我们通过最简单GenerateSimpleSprite()函数来了解Image如何绘制Simple类型的图片的：
```
void GenerateSimpleSprite(VertexHelper vh, bool lPreserveAspect)
{
    Vector4 v = GetDrawingDimensions(lPreserveAspect);
    var uv = (activeSprite != null) ? Sprites.DataUtility.GetOuterUV(activeSprite) : Vector4.zero;

    var color32 = color;
    vh.Clear();
    vh.AddVert(new Vector3(v.x, v.y), color32, new Vector2(uv.x, uv.y));
    vh.AddVert(new Vector3(v.x, v.w), color32, new Vector2(uv.x, uv.w));
    vh.AddVert(new Vector3(v.z, v.w), color32, new Vector2(uv.z, uv.w));
    vh.AddVert(new Vector3(v.z, v.y), color32, new Vector2(uv.z, uv.y));

    vh.AddTriangle(0, 1, 2);
    vh.AddTriangle(2, 3, 0);
}
```
其中用[VertexHelper](https://docs.unity3d.com/2017.4/Documentation/ScriptReference/UI.VertexHelper.html)的两个方法AddVert()来AddTriangle()绘制mesh。  
方法|说明|
:---|:---|
AddVert|Add a single vertex to the stream.
AddTriangle|Add a triangle to the buffer.

AddVert(Vector3 position, Color32 color, Vector2 uv0)用来添加要绘制的Mesh的顶点信息，包含顶点坐标、顶点颜色、顶点UV。  
AddTriangle(int idx0, int idx1, int idx2)用来设置三角面的顶点索引。  
从GenerateSimpleSprite()代码发现，顶点坐标是从GetDrawingDimensions()中获取的。UV坐标则是用Sprites.DataUtility.GetOuterUV(activeSprite)来获取的  
先来分析下GetDrawingDimensions()函数：
```
/// Image's dimensions used for drawing. X = left, Y = bottom, Z = right, W = top.
private Vector4 GetDrawingDimensions(bool shouldPreserveAspect)
{
    var padding = activeSprite == null ? Vector4.zero : Sprites.DataUtility.GetPadding(activeSprite);
    var size = activeSprite == null ? Vector2.zero : new Vector2(activeSprite.rect.width, activeSprite.rect.height);

    Rect r = GetPixelAdjustedRect();

    int spriteW = Mathf.RoundToInt(size.x);
    int spriteH = Mathf.RoundToInt(size.y);

    var v = new Vector4(
            padding.x / spriteW,
            padding.y / spriteH,
            (spriteW - padding.z) / spriteW,
            (spriteH - padding.w) / spriteH);

    if (shouldPreserveAspect && size.sqrMagnitude > 0.0f)
    {
        var spriteRatio = size.x / size.y;
        var rectRatio = r.width / r.height;

        if (spriteRatio > rectRatio)
        {
            var oldHeight = r.height;
            r.height = r.width * (1.0f / spriteRatio);
            r.y += (oldHeight - r.height) * rectTransform.pivot.y;
        }
        else
        {
            var oldWidth = r.width;
            r.width = r.height * spriteRatio;
            r.x += (oldWidth - r.width) * rectTransform.pivot.x;
        }
    }

    v = new Vector4(
            r.x + r.width * v.x,
            r.y + r.height * v.y,
            r.x + r.width * v.z,
            r.y + r.height * v.w
            );

    return v;
}
```
GetDrawingDimensions()返回一个Vector4，4个值分别是左、下、右、上边界的值。函数里也用到了Sprite.DataUtility这个工具类，之前在获取UV的时候也用到了。它包括4个静态方法：
方法|说明|
:---|:---|
GetInnerUV|Inner UV's of the Sprite.
GetOuterUV|Outer UV's of the Sprite.
GetMinSize|Minimum width and height of the Sprite.
GetPadding|Return the padding on the sprite.

- GetPadding： 获取padding，如果sprite的纹理来自图集，那么当其被打到图集里时可能四边会被裁掉一些透明区域，padding即表示上下左右四边被裁掉的区域的大小，单位为像素；
- GetInnerUV： 获取内圈UV（纹理坐标），在Sliced模式中会用到；
- GetOuterUV： 获取外圈UV（纹理坐标）；
- GetMinSize： 获取最小宽高尺寸；

接着activeSprite.rect包含Sprite的原始纹理的坐标与尺寸。
还用到了GetPixelAdjustedRect()，定义在抽象类Graphic中，用于获取像素修正过的Rect：
```
public Rect GetPixelAdjustedRect()
{
    if (!canvas || canvas.renderMode == RenderMode.WorldSpace || canvas.scaleFactor == 0.0f || !canvas.pixelPerfect)
        return rectTransform.rect;
    else
        return RectTransformUtility.PixelAdjustRect(rectTransform, canvas);
}
```
GetDrawingDimensions()函数中的计算就是为了获取去掉Padding之后的顶点坐标，Size是纹理的宽高尺寸（包括Padding），r是RectTransform的尺寸。  
最后，在得到去掉Padding之后的顶点坐标，以及外圈的UV坐标之后，就可以通过VertexHelper来绘制图片了。**由此给我们实现圆形裁剪和方形裁剪提供了思路，可以通过自定义顶点坐标，来绘制圆形和方形的网格，在通过自定义UV坐标来裁剪原图需要显示的图案**  
# **实现方形裁剪**
下面先从简单一些的方形裁剪开始，首先来确定方形的顶点坐标，我们可以通过GetPixelAdjustedRect()函数来获取Image组件的矩形大小，再根据宽高大小计算出方形的左下右上的顶点坐标(PS:这里不用计算Padding的大小)，代码如下：
```
Rect r = GetPixelAdjustedRect();
float v_y = r.y;
float v_w = r.y;
float v_x = r.x;
float v_z = r.x;
float d = r.height-r.width;
if (d>0) // 高大于宽
{
    float dy = d*0.5f;
    v_y += dy;
    v_w += r.width + dy;
    v_z += r.width;
}
else
{
    float dx = -d*0.5f;
    v_x += dx;
    v_z += r.height + dx;
    v_w += r.height;
}
```
接下来截取方形的UV坐标，计算方法和顶点坐标类似，也是判断宽高的大小，然后去计算UV方形的左下右上点的坐标，但需要额外考虑一点就是截图的锚点设置，锚点枚举如下：
```
public enum ShapeAnchors
{
    Center,
    Top,
    Bottom,
    Left,
    Right,
    TopLeft,
    TopRight,
    BottomLeft,
    BottomRight,
}
```
计算代码如下：
```
float tw = activeSprite.texture.width;
float th = activeSprite.texture.height;
float s = tw / th;
float dx = (uv.z - uv.x) * s;
float dy = (uv.w - uv.y);
float d = dx - dy;//d>0 宽大于高，d<0 高大于宽
if (d < 0)
{
    if (shapeAnchorsType == ShapeAnchors.Top || shapeAnchorsType == ShapeAnchors.TopLeft || shapeAnchorsType == ShapeAnchors.TopRight)
    {
        uv.y -= d;
    }
    else if (shapeAnchorsType == ShapeAnchors.Center || shapeAnchorsType == ShapeAnchors.Left || shapeAnchorsType == ShapeAnchors.Right)
    {
        uv.y -= d / 2;
        uv.w += d / 2;
    }
    else if (shapeAnchorsType == ShapeAnchors.Bottom || shapeAnchorsType == ShapeAnchors.BottomLeft || shapeAnchorsType == ShapeAnchors.BottomRight)
    {
        uv.w += d;
    }
}
else
{
    if (shapeAnchorsType == ShapeAnchors.Center || shapeAnchorsType == ShapeAnchors.Top || shapeAnchorsType == ShapeAnchors.Bottom)
    {
        uv.x += d / 2;
        uv.z -= d / 2;
    }
    else if (shapeAnchorsType == ShapeAnchors.Left || shapeAnchorsType == ShapeAnchors.TopLeft || shapeAnchorsType == ShapeAnchors.BottomLeft)
    {
        uv.z -= d;
    }
    else if (shapeAnchorsType == ShapeAnchors.Right || shapeAnchorsType == ShapeAnchors.TopRight || shapeAnchorsType == ShapeAnchors.BottomRight)
    {
        uv.x += d;
    }
}
```
得到顶点坐标和UV坐标之后，我们就可以画出我们想要的方形图形了。但是先别着急，还可以再做点优化，添加一些自定义参数来调整裁剪的效果，比如说裁剪区域的缩放[SetShapeScale()]，还有裁剪区域的偏移[SetShapeAnchorsOffset()]，以及计算锚点时，是否考虑Padding的影响[IncludePaddingUV()]。这里就不一一列举代码了，有兴趣可以查看[LImage.cs](https://github.com/lyback/UGUI_Extension/blob/master/Assets/Scripts/UI/Core/LImage.cs)  
绘制方形裁剪的完整代码如下：
```
/// <summary>
/// 绘制方形图片
/// </summary>
void GenerateSimpleSquareSprite(VertexHelper vh, bool lPreserveAspect)
{
    Sprite activeSprite = this.overrideSprite;
    Color32 color = this.color;
    vh.Clear();
    var uv = (activeSprite != null) ? Sprites.DataUtility.GetOuterUV(activeSprite) : Vector4.zero;
    uv = SetShapeAnchors(uv);
    uv = SetShapeScale(uv);
    uv = SetShapeAnchorsOffset(uv);
    if (m_ShapeAnchorsCalPadding)
    {
        uv = IncludePaddingUV(uv);
    }
    //不计算padding
    Rect r = GetPixelAdjustedRect();
    float v_y = r.y;
    float v_w = r.y;
    float v_x = r.x;
    float v_z = r.x;
    float d = r.height-r.width;
    if (d>0) // 高大于宽
    {
        float dy = d*0.5f;
        v_y += dy;
        v_w += r.width + dy;
        v_z += r.width;
    }
    else
    {
        float dx = -d*0.5f;
        v_x += dx;
        v_z += r.height + dx;
        v_w += r.height;
    }

    vh.AddVert(new Vector3(v_x, v_y), color, new Vector2(uv.x, uv.y));
    vh.AddVert(new Vector3(v_x, v_w), color, new Vector2(uv.x, uv.w));
    vh.AddVert(new Vector3(v_z, v_w), color, new Vector2(uv.z, uv.w));
    vh.AddVert(new Vector3(v_z, v_y), color, new Vector2(uv.z, uv.y));

    vh.AddTriangle(0, 1, 2);
    vh.AddTriangle(2, 3, 0);
}
```
# **实现圆形裁剪**
圆形裁剪和方形裁剪实现思路是一样的，差别在于顶点坐标和UV坐标的算法不同，圆形的坐标计算稍微复杂一些。下面直接上代码：
```
/// <summary>
/// 绘制圆形图片
/// </summary>
void GenerateSimpleCircleSprite(VertexHelper vh, bool lPreserveAspect)
{
    Sprite activeSprite = this.overrideSprite;
    Color32 color = this.color;
    vh.Clear();
    var uv = (activeSprite != null) ? Sprites.DataUtility.GetOuterUV(activeSprite) : Vector4.zero;
    uv = SetShapeAnchors(uv);
    uv = SetShapeScale(uv);
    uv = SetShapeAnchorsOffset(uv);
    if (m_ShapeAnchorsCalPadding)
    {
        uv = IncludePaddingUV(uv);
    }

    float tw = rectTransform.rect.width;
    float th = rectTransform.rect.height;
    float outerRadius = tw > th ? th / 2f : tw / 2f;
    float uvCenterX = (uv.x + uv.z) * 0.5f;
    float uvCenterY = (uv.y + uv.w) * 0.5f;
    float uvScaleX = (uv.z - uv.x) / tw;
    float uvScaleY = (uv.w - uv.y) / th;
    float degreeDelta = 2 * Mathf.PI / circleShape_Segements;
    int curSegements = (int)(circleShape_Segements * circleShape_FillPercent);

    float curDegree = 0f;
    int verticeCount = curSegements + 1;
    //圆心
    Rect r = GetPixelAdjustedRect();
    Vector3 centerVertice = new Vector3(r.x + r.width / 2f, r.y + r.height / 2f);
    vh.AddVert(centerVertice, color, new Vector2(uvCenterX, uvCenterY));
    Vector3 curVertice = new Vector3();
    for (int i = 1; i < verticeCount; i++)
    {
        float cosA = Mathf.Cos(curDegree);
        float sinA = Mathf.Sin(curDegree);
        float posX = cosA * outerRadius;
        float posY = sinA * outerRadius;
        curVertice.x = posX + centerVertice.x;
        curVertice.y = posY + centerVertice.y;
        vh.AddVert(curVertice, color, new Vector2(uvCenterX + posX * uvScaleX, uvCenterY + posY * uvScaleY));
        curDegree += degreeDelta;
    }
    int triangleCount = curSegements * 3;
    for (int i = 0, vIdx = 1; i < triangleCount - 3; i += 3, vIdx++)
    {
        vh.AddTriangle(vIdx, 0, vIdx + 1);
    }
    if (circleShape_FillPercent == 1)
    {
        //首尾顶点相连
        vh.AddTriangle(verticeCount - 1, 0, 1);
    }
}
```

最后附上[Github项目地址](https://github.com/lyback/UGUI_Extension)