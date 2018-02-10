<h1 align="center">Wind Shader</h1>

<h3 align="center">Written by @XorDev</h3>
<h3 align="center">http://xorshaders.weebly.com/tutorials/8-wind-shader</h3>

To make our wind, we need a simple ripple effect. We will create a vec2 and call it "Coord". This will be used for the wind distortion. We will set it to "v_vTexcoord". Now you need to change the texture2D coordinates to our Coord vector. It should look like this:
 
    vec2 Coord = v_vTexcoord;
    gl_FragColor = v_vColour * texture2D( gm_BaseTexture, Coord);

This is a pass-through shader. If you run the shader applied to a sprite, it shouldn't change anything. To make a ripple we will use a cosine wave. We could do it like this:

    vec2 Coord = v_vTexcoord + vec2(cos(v_vTexcoord.y*30.0)/30.0,0);

If you run this you will see this (before and after):

This is closer to what we want, but we need it to be animated. This works by shifting the X coordinates in a cosine wave shape. This makes a nice ripple. You can adjust the frequency by changing the first 30.0 and adjust the amplitude by changing the second 30.0. We'll come back to this at the end of the tutorial

<h3 align="center">Animating</h3>

We need to add a uniform float and we'll call it "Time" and we'll add it in the cosine like this:

    vec2 Coord = v_vTexcoord + vec2(cos(v_vTexcoord.y*30.0+Time*6.2831)/30.0,0);

It is multiplied by 6.2831 (pi times two) because cosine uses angles in radians so that it does a full cycle every second.
We'll set the time uniform in GameMaker like this in the draw event:

    var uTime = shader_get_uniform(shdr_wind,"Time");

    shader_set(shdr_wind)
    shader_set_uniform_f(uTime,current_time/1000)
    draw_self()
    shader_reset()

This will set "Time" to the number of seconds since the game started. If you run this it will be animated. You may notice a problem where the base of the grass moves just as much as the top.

<h3 align="center">Fixing And Perfecting</h3>

So to fix the problem I just mentioned we will multiply cosine by the" (1.0-v_vTexcoord.y)". This will make the distortion decrease as it approaches the bottom of the sprite. Here is what I did:

    vec2 Coord = v_vTexcoord + vec2(cos(v_vTexcoord.y*30.0 + Time*6.2831)/30.0,0)*(1.0-v_vTexcoord.y);

It looks much better now. This shader is currently proportion to the sprite size. If it's long it will stretch the waves, but if it's short it will shrink the waves. That is because the waves are based off of the v_vTexcoord which is between 0 and 1. To fix this we can use the position attribute from the vertex shader because it uses the vertex coordinates rather than the texture coordinates.
 We need to add a varying vec2 called "v_vPosition" in the vertex and fragment shaders by the other varying vectors. Set it in the vertex shader like this:

    v_vPosition = in_Position.xy;

And back to the fragment shader use this for the coordinates:

    vec2 Size = vec2(256,128);
    vec2 Wave = vec2(48,5);

    vec2 Coord = v_vTexcoord + vec2(cos((v_vPosition.y/Wave.x+Time)*6.2831)*Wave.y,0)/Size*(1.0-v_vTexcoord.y)​;

"Size" should be the size of the texture. Use a uniform if necessary. "Wave" is for adjusting the wave shape. The first number is the height of the wave and the second number is the amplitude. This works by making a cosine wave of the chosen size with time added (multiplied by 6.2831 because of radians) and then multiplied by the amplitude; then divided by "Size" to get it between 0 and 1 and finally multiplied by the code I mentioned above to fix the bottom.
