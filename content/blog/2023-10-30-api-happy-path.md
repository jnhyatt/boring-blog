+++
title = "Dev Log #2"
[taxonomies]
tags = [ "dev log", "chain reaction" ]
+++

#

![@/icon.png](josh)

I seem to spend a lot of time beating my head against walls. I'm not sure if it's because I don't do things the "right" way or maybe because I'm always trying to do something *weird*. Whatever the reason, I feel like I'm always mucking around some dark corner of whatever API it is I'm trying to bully into submission rather than taking the API happy path.

Just as a simple example, I'm building a game board select page for Chain Reaction. Originally I had a carousel that you could swipe through to select the board you wanted, but over time I've learned that 1) Android and carousels aren't friends, and 2) users want to see all their options at a glance. I redesigned it into a two-column grid view with tiles for each board. Now you simply scroll through the list and click the one you want. Now you can see 4-6 boards at a time and the interface is much easier to understand the first time you see it.

For the grid view, `RecyclerView` seems to be the go-to tool for all things list- and grid-related. For the preview on the tile, I already have logic that renders a board to a `SurfaceView` with OpenGL, but `SurfaceView` is not the kind of view you want to have a lot of. As I understand it, the surface it manages is handed directly to the Android compositor and can be very limited both in layout capabilities as well as number of available simultaneous views. I consider `TextureView` to be somewhat of a "direct" replacement for it -- performance is not as good since the result needs to go through several layers of compositing/laying out, but it behaves much more like a typical Android view while also giving you a surface to draw to. Additionally, some wonderful human on GitHub has provided a [gist with a `GLTextureView` implementation](https://gist.github.com/wasabeef/ac7454ed01deb99afb6c). It's more or less a `GLSurfaceView` with all references to `SurfaceView` replaced with `TextureView`. It manages initializing EGL for you and even spools up a rendering thread.

With me so far? I have `RecyclerView`, and I'm happy to stay on its API happy path. I have `GLTextureView` which admittedly strays a little bit from what I'd call the happy path, but I've also used `GLSurfaceView` so many times my eyes start to bleed when I see `glView.setEGLContextClientVersion(3); glView.setRenderer(myRenderer);`. Now let's put them together and see how things start to break down.

I find `GLTextureView` to be a bit heavy when we're using so many, because I don't see the need to spool up a thread *and* EGL context per tile, especially since on something like a tablet I could have a good dozen boards or more on screen at any given time. However, I'm a great believer in taking the simple route until it proves a problem, so I went ahead.

My understanding of `RecyclerView` is that it would create several of my `BoardPreviewTile`s using `onCreateViewHolder`, bind them using `onBindViewHolder`, and then as those scrolled off the page, it would unbind them but not destroy them. That way when more views scroll onto the screen, it could simply rebind them and stick them back into the view hierarchy. Since `GLTextureView.setRenderer` can only be called once, I figured I could set the renderer in `onCreateViewHolder`, then in `onBindViewHolder` I could actually assign the meshes/shaders used for rendering. This worked fine for the first scroll-past, but if you scroll a view off the screen, then scroll it back on, it stops working.
