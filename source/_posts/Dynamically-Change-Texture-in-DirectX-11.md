title: Dynamically Change Texture in DirectX 11 (SharpDX)
date: 2015-01-15 12:35:05
categories:
- Study
- Graphics
tags:
- DX11
- SharpDX
- C#
---

I had some experience of DX11 compute shader programming when I worked on my undergraduate project.
And recently there is a project to show an object with CG animation mapped on its surface.

My first trial is to change texture's content to frames captured by Kinect camera. There is a great library named [SharpDX](sharpdx.org) which enables DirectX programming in C#.

I've searched on Internet for solution and referred to documentation. Since there lacks a detailed solution or example, I would like to share my understanding in this post.

The idea to show kinect depth image is trivial: just draw a rectangle and bind the image as texture to it. The depth image is changed in kincet callback function, and we update that data to the texture so that kinect camera video is shown on screen.

Based on this idea, we could replace that video with any data we want. Following code show detailed instructions.

###Key Steps:
1. Create a texture with the same size as depth frame. The usage and CPU access flag need to be set available.
{% code lang:csharp %}
var texture = new Texture2D(device, new Texture2DDescription()
{
    Format = Format.R8G8B8A8_UNorm,
    ArraySize = 1,
    MipLevels = 1,
    Width = kinectHelper.DepthFrameDescription.Width,
    Height = kinectHelper.DepthFrameDescription.Height,
    SampleDescription = new SampleDescription(1, 0),
    Usage = ResourceUsage.Dynamic,         // dynamic means texture content could be changed
    BindFlags = BindFlags.ShaderResource,
    CpuAccessFlags = CpuAccessFlags.Write, // cpu has the access to write data
    OptionFlags = ResourceOptionFlags.None,
});
{% endcode %}
<!-- more -->

2. Create a DataStream as a buffer to map data in texture.
{% code lang:csharp %}
// the size of stream is 4 byte(RGBA) * textureSize
DataStream stream = new DataStream(4 * kinectHelper.DepthFrameDescription.Width * kinectHelper.DepthFrameDescription.Height, true, true);
{% endcode %}

3. Use Map and Unmap to change data in texture.
{% code lang:csharp %}
// use MapMode.WriteDiscard to refresh all data in texture
DataBox databox = context.MapSubresource(texture, 0, MapMode.WriteDiscard, MapFlags.None, out stream);
if (!databox.IsEmpty && kinectHelper.depthPixels != null)
    // copy data from kinect to texture
    stream.Write(kinectHelper.depthPixels, 0, 4 * kinectHelper.DepthFrameDescription.Width * kinectHelper.DepthFrameDescription.Height);
context.UnmapSubresource(texture, 0);
{% endcode %}

###ScreenShots:
![](http://tecton.qiniudn.com/kinect_depth_map_screenshot1.png)
![](http://tecton.qiniudn.com/kinect_depth_map_screenshot2.png)

###Demo Source Code:
It is modified on the example in SharpDX and Kinect SDK v2.
You need to install those two libraries before running.
Here is the [github address](https://github.com/tecton/Kinect2-SharpDX-Demo/).