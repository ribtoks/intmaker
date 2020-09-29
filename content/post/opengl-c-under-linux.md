---
title: 'Simple OpenGL program on C# under Linux'
date: 2013-06-19T23:12:44+00:00
author: "Taras Kushnir"
categories:
  - Programming
keywords:
  - c++
  - csharp
  - linux
  - mono
  - opengl
  - opentk
aliases:
  - /2013/simple-opengl-program-on-c-under-linux
---
Today I'll tell you how to write a very simple OpenGL program on C# at Mono platform. Finally we're going to get something like this:<figure id="attachment_1031" class="thumbnail wp-caption aligncenter" style="width: 310px">

[<img class=" wp-image-1031 " src="http://code.jamming.com.ua/wp-content/uploads/2013/06/opentk_test.png?w=300" alt="OpenTK Game Window" width="300" height="241" srcset="http://code.jamming.com.ua/wp-content/uploads/2013/06/opentk_test.png 400w, http://code.jamming.com.ua/wp-content/uploads/2013/06/opentk_test-300x242.png 300w" sizes="(max-width: 300px) 100vw, 300px" />](http://code.jamming.com.ua/wp-content/uploads/2013/06/opentk_test.png)<figcaption class="caption wp-caption-text">Result of OpenGL app</figcaption></figure> 

<!--more-->

Several companies and organizations have developed their own collections of bindings for OpenGL under .NET (Mono), the leader of which is the Tao Framework. The Tao Framework for .NET is a collection of bindings to facilitate cross-platform media application development utilizing the .NET and Mono platforms. But I failed to use it properly under Linux, so I found Open ToolKit, that is a free, cross-platform OpenGL and OpenAL wrapper for C# and other .NET languages.

To use it you should download it from <a href="http://www.opentk.com/" target="_blank">official website of openTK</a> and extract from archive to your favorite location. Create empty project in your favorite IDE. In Linux (currently OpenSUSE 12.3) I use MonoDevelop. In your project References section right click and add reference to _OpenTK.dll_ which you can find via _opentk/Binaries/OpenTK/Release_ path (and also add _System.Drawing.dll_ assembly reference for _Color_ stuff).

Now add some .cs file to your solution and paste this code:

```csharp
using System;

using OpenTK;
using OpenTK.Graphics.OpenGL;
using OpenTK.Graphics;
using OpenTK.Input;

namespace OpenGLStart
{
	public class SimpleWindow : GameWindow
	{
		public SimpleWindow()
			: base(400, 300)
		{
		}

		protected override void OnUpdateFrame (FrameEventArgs e)
		{
			base.OnUpdateFrame (e);
		}

		protected override void OnRenderFrame (FrameEventArgs e)
		{
			GL.Clear(ClearBufferMask.ColorBufferBit);

			// draw something simple using OpenGL

			// ...

			GL.Begin(BeginMode.Triangles);

			// red apex
			GL.Color3(1.0, 0.0, 0.0);
			GL.Vertex2(-1.0, -1.0);

			// green apex
			GL.Color3(0.0, 1.0, 0.0);
			GL.Vertex2(1.0, -1.0);

			// blue apex
			GL.Color3(0.0, 0.0, 1.0);
			GL.Vertex2(0, 1.0);

			GL.End();
			this.SwapBuffers();

		}

		protected override void OnLoad (EventArgs e)
		{
			GL.ClearColor(Color4.RoyalBlue);
		}

		protected override void OnResize (EventArgs e)
		{
			GL.Viewport(0, 0, Width, Height);
			GL.MatrixMode(MatrixMode.Projection);
			GL.LoadIdentity();
			GL.Ortho(-1.0, 1.0, -1.0, 1.0, 0.0, 4.0);
			base.OnResize (e);

		}

		[STAThread]
		public static void Main(string[] args)
		{
			using (var SimpleWindow1 = new SimpleWindow())
			{
				SimpleWindow1.Run();
			}
		}
	}
}
```

Code is quite self-explanatory. We use code in _OnRenderFrame_ to draw main picture and some helpful stuff like _OnResize_ to handle window resizing well. _Main()_ method just creates the window and runs it. Now you can run it and get window with colored triangle.
