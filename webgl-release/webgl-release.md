One of the Qt 5.12 new features is <b>Qt Quick WebGL</b> platform plugin (<i>also known as <b>WebGL streaming</b></i>). It was actually available as a technology preview from Qt 5.10 already, but starting with Qt 5.12 it is a released feature.

<ul>
    <li><a href="#tldr">TLDR</a></li>
    <li><a href="#intro">Intro</a></li>
    <li><a href="#what-is-webgl-streaming">What is WebGL streaming</a></li>
    <li><a href="#how-to-use-it">How to use it</a></li>
    <li><a href="#some-demos">Some demos</a></li>
    <ul style="margin-bottom:0px">
        <li><a href="#demo-device-information">Device Information</a></li>
        <li><a href="#demo-camera">Camera</a></li>
    </ul>
    <li><a href="#use-cases">Use cases</a></li>
    <ul style="margin-bottom:0px">
        <li><a href="#webgl-streaming-vs-actual-web">WebGL streaming vs actual web</a></li>
    </ul>
    <li><a href="#licensing-pricing">Licensing/pricing</a></li>
    <li><a href="#conclusion">Conclusion</a></li>
</ul>

<h3><a name="tldr">TLDR</a></h3>

<pre class="brush:shell">$ ./your-qt-application -platform webgl:port=8998
</pre>

<h3 style="margin-top:30px"><a name="intro">Intro</a></h3>

If you missed previous blog-posts, here they are:

<ul>
    <li><a href="http://blog.qt.io/blog/2017/02/22/qt-quick-webgl-streaming/">Qt Quick WebGL Streaming</a>;</li>
    <li><a href="http://blog.qt.io/blog/2017/03/24/webgl-streaming-raspberry-pi-zero-w/">WebGL streaming in a Raspberry PI Zero W</a>;</li>
    <li><a href="http://blog.qt.io/blog/2017/07/07/qt-webgl-streaming-merged/">Qt WebGL Streaming merged</a>;</li>
    <li><a href="http://blog.qt.io/blog/2017/11/14/qt-webgl-cinematic-experience/">Qt WebGL: Cinematic Experience</a>.</li>
</ul>

There is also a <a href="https://www.ics.com/blog/whats-new-qt-510">good article</a> by Jeff Tranter from ICS.

This post is intended to be a kind of an "unboxing" experience from the perspective of an average Qt "user" - I never tried Qt Quick WebGL streaming myself, neither did I participate in its development.

<h3><a name="what-is-webgl-streaming">What is WebGL streaming</a></h3>

If you read past blog-posts, you can just skip this section. I would however recommend to at least read the <a href="https://doc.qt.io/qt-5/webgl.html">documentation</a>.

WebGL streaming is a <a href="http://doc.qt.io/qt-5/qpa.html">QPA plugin</a> that sends ("streams") OpenGL calls of your Qt Quick application over the network and in turn those are translated into <a href="https://en.wikipedia.org/wiki/WebGL">WebGL</a> calls and thus can be rendered at <a href="https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API">HTML5 Canvas</a>. What it means in practice is that you can have an application running on a remote host and render its GUI in a local web-browser.

Here's how it looks schematically:

<img class="aligncenter" src="https://qt-blog-uploads.s3.amazonaws.com/wp-content/uploads/2018/11/how-it-works.png" title="Qt WebGL, how it works" />

Here's also a <a href="https://youtu.be/X1iDlE06xdA">video</a> from KDE Akademy with a more detailed explanation by <a href="http://blog.qt.io/blog/author/jesusfernandez/">Jesus Fernandez</a>.

But since I'm a simple Qt "user", I don't really care about any of that (<i>it's all hidden from me anyway</i>), and to me everything looks like this:

<img class="aligncenter" src="https://qt-blog-uploads.s3.amazonaws.com/wp-content/uploads/2018/11/how-it-works-simplified.png" title="Qt WebGL, how it works, simplified" />

So I can have a Qt-based application running on some device and work with it from Safari on my iPad. Sounds alright.

Naturally, instead of a device (<i>Raspberry Pi in this case</i>) there could be a desktop computer "hosting" the application, but I think WebGL streaming will be used mostly on embedded platforms (<i>see <a href="#use-cases">use cases</a> section</i>).

Let's now highlight a couple of points we've just learnt about the feature:

<ul>
    <li>The application itself <b>does not</b> run inside a web-browser. Web-browser <b>only renders</b> its GUI;</li>
    <li>So it is <b>neither</b> video-streaming, <b>nor</b> mirroring. It is about "decoupling" application's GUI and showing it in a web-browser;</li>
    <li>Since it's for OpenGL (ES) things only, WebGL streaming does not work with Widgets or any other non-OpenGL stuff.</li>
</ul>

In fact, if you try to launch some "non-compatible" Qt application using WebGL QPA, most likely you'll get the following error:

<pre class="brush:bash">qt.qpa.webgl: WebGL QPA platform plugin: Raster surfaces are not supported
</pre>

<h3 style="margin-top:30px"><a name="how-to-use-it">How to use it</a></h3>

You only need to install it:

<img class="aligncenter" src="https://qt-blog-uploads.s3.amazonaws.com/wp-content/uploads/2018/11/qt-installer-webgl.png" title="Qt WebGL, installation" />

...or, if you are not into installers, build Qt from <a href="http://code.qt.io/cgit/qt/qt5.git/">sources</a> as usual - no special configuration options needed. Actually, with earlier versions <code>-opengl es2</code> option was required, but there is no need in that as <a href="http://doc.qt.io/qt-5/qtquick-visualcanvas-scenegraph.html">Qt Quick Scene Graph</a> can use ES subset even if there is a later version of OpenGL available.

Having installed Qt itself, build any Qt Quick application of yours and launch it with the following command line arguments:

<pre class="brush:bash">$ ./your-qt-application -platform webgl
</pre>

Yes, you don't need to make any modifications in your source code, it just works. Open the following address in your web-browser: <code>127.0.0.1:8080</code>, where <code>127.0.0.1</code> is the IP address of the host running your application.

If you want to use a different port, you can specify it like that:

<pre class="brush:bash">$ ./your-qt-application -platform webgl:port=8998
</pre>

Needless to say, Qt WebGL is cross-platform, and it works equally fine on Linux, Mac OS and Windows. Although, there are some differences in launching applications:

Linux:
 
<pre class="brush:bash">./your-qt-application -platform webgl:port=8998
</pre>

Mac OS:

<pre class="brush:bash">QT_QPA_PLATFORM=webgl:port=8998 ./your-qt-application.app/Contents/MacOS/your-qt-application
</pre>

...because Qt version I have at the moment apparently ignores <code>-platform</code> option (<i>which sounds like a bug that needs to be reported</i>).

Windows:

<pre class="brush:bash">your-qt-application.exe -platform webgl:port=8998
</pre>

And of course you can do it in a cross-platform (<i>duh</i>) way via <a href="http://doc.qt.io/qt-5/qtglobal.html#qputenv">qputenv()</a> in your <code>main.cpp</code>:

<pre class="brush:cpp">// ...
qputenv("QT_QPA_PLATFORM", "webgl:port=8998");

QGuiApplication app(argc, argv);
// ...
</pre>

Speaking about web-browsers support, I tried several ones (<i>except for the one with trident and his younger brother</i>), and it worked in all of them, so it looks like WebGL is well supported in modern browsers nowadays. Although, I did experience a couple of occasional page reloads, and also Chrome on Android tablet even crashed once, so apparently not that well, but that's really outside the Qt's scope.

With regards to performance, the most "busy" time is during the initialization phase, when web-browser is receiving buffers, textures, glyphs, atlases and so on. After the first draw call, the bandwidth usage is pretty low. And by the way, since OpenGL ES calls are sent as binary data, it should be more "light-weight" than VNC. I am actually thinking about writing another blog-post to compare WebGL streaming and VNC in terms of network utilization.

<h3><a name="some-demos">Some demos</a></h3>

There is already a fair amount of demo videos in previous blog-posts, and here's also another nice <a href="https://youtu.be/C4KSfQ3XFC8">compilation</a>, so I decided to create a couple of my own.

<h4><a name="demo-device-information">Device Information</a></h4>

This one is a rather simple demo application. It gathers some information about the platform it is running on. For example, here's what it shows when I run it on my Mac:

<div style="text-align:center;font-size:12px;margin-bottom:15px">[video src="https://git.qt.io/arsidyak/webgl-release/raw/master/webgl-release-blog/img/di.mp4" autoplay="on" loop="on"]
If video doesn't play in your browser, you can download it <a href="https://git.qt.io/arsidyak/webgl-release/raw/master/webgl-release-blog/img/di.mp4">here</a></div>

Let's now run it on Raspberry Pi using WebGL streaming plugin, connect to it from a web-browser on the same Mac and ascertain that it no longer reports Mac OS as operating system and that platform is <b>webgl</b> now. There is one more thing to see here - pay attention to changing values of the "screen" resolution:

<div style="text-align:center;font-size:12px;margin-bottom:15px">[video class="aligncenter" src="https://git.qt.io/arsidyak/webgl-release/raw/master/webgl-release-blog/img/webgl-different-canvas-sizes.mp4" loop="on"]
If video doesn't play in your browser, you can download it <a href="https://git.qt.io/arsidyak/webgl-release/raw/master/webgl-release-blog/img/webgl-different-canvas-sizes.mp4">here</a></div>

As you can see, when I resize the browser window, application (<i>beside nicely adapting its layout</i>) reports changed screen resolution values. This is because it takes canvas dimensions for the screen resolution. Let's check that in web-browser inspector:

<img class="aligncenter" src="https://qt-blog-uploads.s3.amazonaws.com/wp-content/uploads/2018/11/webgl-canvas-equals-screen.png" title="Qt WebGL, canvas equals screen" />

By the way, we can take a look at user input events here as well:

<img class="aligncenter" src="https://qt-blog-uploads.s3.amazonaws.com/wp-content/uploads/2018/11/webgl-js-events.png" title="Qt WebGL, JS events" />

So application does indeed run on Raspberry, and what I have in my browser is just its "streamed" GUI.

<h4><a name="demo-camera">Camera</a></h4>

This demo is a bit more practical one - it's a camera controlled by robotic-ish arm which is mounted on a Raspberry Pi device:

<img class="aligncenter" src="https://qt-blog-uploads.s3.amazonaws.com/wp-content/uploads/2018/11/rpi-camera-pimoroni-hat.jpg" title="Raspberry Pi with camera and Pimoroni pan-tilt HAT" />

The idea is to control the camera (its pan and tilt) with a Qt-based application running on the device, but to do that remotely from a web-browser on some tablet. And of course we would like to see what cameras is looking at (<i>its viewfinder</i>). And it also would be nice to make photos with the camera.

Here's a list of required hardware for such a setup:

<ul>
    <li><a href="https://thepihut.com/products/pan-tilt-hat">Pan-Tilt HAT</a>;</li>
    <li><a href="https://thepihut.com/products/raspberry-pi-3-model-b-plus">Raspberry Pi 3 Model B+</a>. I could choose RPi Zero as well, but this HAT fits better on a "full-sized" RPi;</li>
    <li><a href="https://thepihut.com/products/pibow-3b-coupe-raspberry-pi-3-3b">Pibow 3B+ Coupe Royale</a>;</li>
    <li><a href="https://thepihut.com/products/raspberry-pi-camera-module">Raspberry Pi Camera Module V2</a>.</li>
</ul>

And that's <a href="https://learn.pimoroni.com/tutorial/sandyj/assembling-pan-tilt-hat">the manual</a> I used to assemble it.

Now let's take a little detour (<i>spoiler: I will be promoting Qt's commercial features</i>). You might have noticed that for <a href="#demo-device-information">Device Information</a> demo I used <a href="https://doc.qt.io/QtForDeviceCreation/qtb2-index.html">Boot to Qt</a> image, saving myself quite some time and efforts with regards to building Qt-based application for Raspberry Pi and deploying it there.

But Boot to Qt is a commercial-only feature, and without it you'll have to go though some more steps setting up system environment. Here's what it takes with a regular <a href="https://www.raspberrypi.org/downloads/raspbian/">Raspbian Stretch Lite</a> image as an example:

<ul>
    <li>Since Qt Multimedia module is used, you need to make sure that <a href="https://en.wikipedia.org/wiki/GStreamer">GStreamer</a> is installed in the system and you have correct plugins available;</li>
    <li>Get the latest Qt build (5.12) for Raspberry Pi. Either set a cross-compilation toolchain on your desktop or build it right on the device. Building Qt from sources directly on Raspberry Pi is actually a viable option (<i>especially if you failed with cross-compilation toolchain</i>), although compilation itself takes around 10 hours and requires some dances with increasing available swap size (<i>1 GB of RAM is really not enough</i>);</li>
    <li>Build <a href="https://en.wikipedia.org/wiki/Video4Linux">V4L</a> driver to make camera discoverable by GStreamer and thus Qt;</li>
    <li>Come up with a convenient way of building/deploying your applications on device.</li>
</ul>

Even though this list of steps is not too long, in practice it can take you up to several days before you get a working setup, whether with Boot to Qt image you get everything working out-of-the-box, and you can run your applications on the connected device right from the Qt Creator. But enough with the promotional part, let's get back to demo.

The GUI layout looks like the following:

<img class="aligncenter" src="https://qt-blog-uploads.s3.amazonaws.com/wp-content/uploads/2018/11/webgl-release-mac-camera.png" title="Qt Multimedia, camera demo application, viewfinder" />

Most of the space on the first tab is taken by the camera's viewfinder, which is implemented with <a href="http://doc.qt.io/qt-5/qml-qtmultimedia-videooutput.html">VideoOutput</a> and <a href="http://doc.qt.io/qt-5/qml-qtmultimedia-camera.html">Camera</a> itself:

<pre class="brush:bash">Camera { id: camera }

VideoOutput {
    anchors.fill: parent
    fillMode: VideoOutput.PreserveAspectCrop
    source: camera
}
</pre>

There are two <a href="https://doc.qt.io/qt-5.11/qml-qtquick-controls2-slider.html">sliders</a>, horizontal and vertical - for controlling pan and tilt of the camera:

<pre class="brush:bash">Slider {
    id: sliderTilt
    orientation: Qt.Vertical
    from: root.maxValue
    value: 0
    stepSize: 1
    to: -root.maxValue

    onPressedChanged: {
        if (!pressed)
        {
            backend.movePanTilt(basePath, sliderPan.value, sliderTilt.value)
        }
    }
}
</pre>

Pan-Tilt HAT servos are interfaced via <a href="https://en.wikipedia.org/wiki/I²C">I2C</a>, and due to the lack of time I went with fast and dirty solution - by using Pimoroni's <a href="https://github.com/pimoroni/pantilt-hat">Python library</a>. At some point I would like to do it properly with C/C++, although it's really not the point of the demo.

There is also a <a href="https://doc.qt.io/qt-5/qml-qtquick-controls2-button.html">button</a> for taking photos using <a href="http://doc.qt.io/qt-5/qml-qtmultimedia-cameracapture.html">CameraCapture</a>:

<pre class="brush:bash">Button {
    scale: hovered ? (pressed ? 0.9 : 1.1) : 1

    background: Rectangle {
        color: "transparent"
    }

    Image {
        anchors.fill: parent
        source: "/img/camera.png"
    }

    onClicked: {
        camera.imageCapture.captureToLocation(basePath + "shots/" + getCurrentDateTime() + ".jpg");
    }
}
</pre>

Second tab contains a list of taken photos:

<img class="aligncenter" src="https://qt-blog-uploads.s3.amazonaws.com/wp-content/uploads/2018/11/webgl-release-mac-photos.png" title="Qt Multimedia, camera demo application, photos" />

...which is implemented by <a href="http://doc.qt.io/qt-5/qml-qt-labs-folderlistmodel-folderlistmodel.html">FolderListModel</a>:

<pre class="brush:bash">ListView {
    FolderListModel {
        folder: "file:" + basePath + "shots/"
        nameFilters: ["*.jpg"]
    }
    
    model: folderModel
    
    delegate: ItemDelegate {
        text: model.fileName
    }
}
</pre>

Full application source code is available <a href="https://git.qt.io/arsidyak/webgl-release/tree/master/webgl-release-demo">here</a>.

Now let's see it in action. There are 3 spirits placed on my table, surrounding the camera, and I want to take photos of each. I built and ran the application on device with <code>-platform webgl</code>, and connected to it over Wi-Fi from Safari on my iPad:

<p style="text-align:center">[embed width="900"]https://youtu.be/7MhMZ3qMQNI[/embed]</p>

As you can see, the plan worked out just fine: seeing camera's viewfinder, I can remotely control its position and take photos of the objects I'm interested in.

<h3><a name="use-cases">Use cases</a></h3>

Most obvious use case for WebGL streaming is the ability to have a decent GUI for some low-end device with limited computing power, without GPU, and quite often without any display at all. For instance, that is a common scenario for industrial automation domain, where you can have lots of headless devices installed all over the factory: they can be distributed over quite a significant area or even mounted in places with hazardous environment - being able to control/configure those remotely comes to be rather handy.

Reading discussion at <a href="https://news.ycombinator.com/item?id=14718043">Hacker News</a>, I stumbled upon a "reverse" idea: what if it's the other way around, what if "device" is actually a very powerful server, and you work with it from your regular desktop. That way you can can perform some heavy calculations on the server while having GUI in your web-browser (<i>which actually begs for an HTML-based frontend but more on this in the <a href="#webgl-streaming-vs-actual-web">next section</a></i>).

Another possible use-case is an anti-piracy measure. Let's say you want to protect your software from being "cracked" or "pirated". Obviously, if there is nothing running on the client, then there is nothing to crack as your users only have GUI rendered in their browsers, and the application itself is running on your server. Sounds interesting, but there are several drawbacks here:

<ul>
    <li>While WebGL streaming performs well in local network, using it over the internet will result in significant latency;</li>
    <li>Connection is not encrypted, so it is not secure;</li>
    <li>Currently only one connection at a time is supported (<i>so only one user</i>).</li>
</ul>

Overall, supporting only one connection at a time fairly reduces the number of possible use cases, and unfortunately it is unlikely that current implementation of the feature will improve in that regard, so it is more of a task for <a href="https://bugreports.qt.io/browse/QTBUG-62425">Qt 6</a>. By the way, there is an idea to complement streaming with an ability of mirroring as in some cases having the latter is more important.

Speaking about mirroring, I would like to mention our recent <a href="https://www.qt.io/events/remote-ui-with-qt-for-automation-on-arm-based-edge-devices-1539882056/">webinar</a> that we had together with Toradex. There you can see an interesting combination of WebGL streaming and <a href="https://doc.qt.io/qt-5/qtremoteobjects-index.html">Remote Objects</a>, which allows you to implement mirroring functionality as of now already.

Another noticeable aspect of WebGL streaming is so-called "zero install" concept - you don't have to install/deploy anything on clients (<i>desktops/tablets/smartphones/etc</i>) as the only thing needed is just a web-browser. However, <a href="http://blog.qt.io/blog/2018/11/19/getting-started-qt-webassembly/">Qt for WebAssembly</a> seems to be a bit more suitable for that purpose.

<h4><a name="webgl-streaming-vs-actual-web">WebGL streaming vs actual web</a></h4>

Some of you might ask, what is the point of relying on WebGL streaming in the first place? Since it's all about web-browser, one can just take a regular web-server and create a web-application - result will be almost the same: backend is hosted on the remote device and HTML-based GUI is rendered in the web-browser.

That is a very good and fair question. I actually have some experience in web-development, so I asked this question myself. Let's try to answer it, hopefully without starting yet another holy war.

Indeed, in some cases it is enough just to have a simple REST API, especially if you only need to get some plain text data values. So it is likely that Qt-based application with WebGL streaming would be an overkill for such purpose.

However, in more sophisticated scenarios (<i>for example, when you need to control some hardware</i>) Qt-based application with WebGL-streamed GUI might fit better, because that way you'll get a powerful backend (<i>C++/Qt</i>), and I would also mention that creating a complex, appealing and performant frontend is (<i>considerably</i>) easier with Qt Quick rather than with HTML/CSS/JS, but this statement does look like a beginning of yet another holy war, so I'll keep that as my personal opinion.

And the last thing worth to mention here - if you already have a Qt-based application, then WebGL streaming is an obvious option, because it will cost you nothing to have a remote GUI for it.

<h3><a name="licensing-pricing">Licensing/pricing</a></h3>

WebGL streaming plugin is available under commercial and Open Source licenses (<i>but GPLv3 only</i>). And for commercial customers it is included in both Application Development and Device Creation products with no additional charge.

<h3><a name="conclusion">Conclusion</a></h3>

So you're now able to use web-browser as a remote GUI client for your Qt Quick applications with no efforts - it only takes one command line parameter.

In terms of further development, I reckon the next thing to be expected is connection security/encryption, both for WebSocket and WebServer. WebSocket part should be pretty straightforward as <a href="http://doc.qt.io/qt-5/qwebsocket.html">QWebSocket</a> already supports secure connection (<code>wss://</code>). And WebServer part, if you remember, from the very beginning was a temporary solution, and research on proper implementation (<i>including support for HTTPS</i>) is still ongoing.

Meanwhile, if you have any other feature-requests or maybe bugs to report, please use our tracker for that: <a href="http://bugreports.qt.io/">http://bugreports.qt.io/</a> (<i>choose <b>QPA: WebGL</b> component</i>). Your feedback will help our product management team to shape the feature's roadmap.
