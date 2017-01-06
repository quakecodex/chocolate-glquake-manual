## Initialization

Quake initializes the renderer in the `Host_Init` function. There's a lot going on, but here is the code that's releveant to the renderer:

_host.c_
```
void Host_Init (quakeparms_t *parms)
{
	// Other initialization code ommiteed for clarity
    
    /* Creates a single checkerboard texture for debugging */
	R_InitTextures ();
    
    /* Sets up a window with a valid OpenGL context */
    VID_Init (host_basepal);
    
    /* Loads in all of the interface textures and sets them up for drawing */
    Draw_Init ();
    
    /* More UI initialization, sets up cvars and loads more textures */
    SCR_Init ();
    
    /* Sets up rendering cvars and inits particle systems */
    R_Init ();
}
```


Four our purposes, `VID_Init` contains the most important code. This where Quake sets up the rendering window. The other functions are farely straightforward.

_gl_vidsdl.c_
```
void VID_Init (unsigned char *palette) 
{
    // Window initialization code omitted.
    
    /* Tell SDL what buffer settings to use with OpenGL */
    SDL_GL_SetAttribute(SDL_GL_RED_SIZE, 8);
    SDL_GL_SetAttribute(SDL_GL_GREEN_SIZE, 8);
    SDL_GL_SetAttribute(SDL_GL_BLUE_SIZE, 8);
    SDL_GL_SetAttribute(SDL_GL_DEPTH_SIZE, 24);
    SDL_GL_SetAttribute(SDL_GL_DOUBLEBUFFER, 1);
    
    /* Create the Window with an OpenGL Context */
	sdlFlags = SDL_OPENGL;
	if (modelist[vid_default].fullscreen)
		sdlFlags |= SDL_FULLSCREEN;
	pBackbuffer = SDL_SetVideoMode(modelist[vid_default].width, 
                                   modelist[vid_default].height, 
                                   modelist[vid_default].bpp, 
                                   sdlFlags);
    if (pBackbuffer == NULL) 
        Sys_Error("Unable to set video mode.");
        
    // Code omitted.
    
    /* Initialize Openl */
	GL_Init ();
    
    // Finishes initializing.
}
```

Then it does the basic OpenGL setup.

_gl_vidsdl.c_
```
void GL_Init (void)
{
    /* Get basic driver information and print it to the console. */
    gl_vendor = (const char*)glGetString (GL_VENDOR);
	Con_Printf ("GL_VENDOR: %s\n", gl_vendor);
	gl_renderer = (const char*)glGetString (GL_RENDERER);
	Con_Printf ("GL_RENDERER: %s\n", gl_renderer);
	gl_version = (const char*)glGetString (GL_VERSION);
	Con_Printf ("GL_VERSION: %s\n", gl_version);
	gl_extensions = (const char*)glGetString (GL_EXTENSIONS);
	// Truncate extenstion string, to prevent buffer overrun
	// Sometimes GL extenstion string doesn't have terminating null.
	Q_strncpy(ext, (char*)gl_extensions, 2048);
	ext[2047] = '\0';
	Con_Printf ("GL_EXTENSIONS: %s\n", ext);

    /* Check for vendor exceptions */
    if (_strnicmp(gl_renderer,"PowerVR",7)==0)
         fullsbardraw = true;
    if (_strnicmp(gl_renderer,"Permedia",8)==0)
         isPermedia = true;

    /* Load in necessary OpenGL Extensions for texturing */
	CheckTextureExtensions ();
	CheckMultiTextureExtensions ();

    /* Set up Quake's rendering states */
	glClearColor (1,0,0,0);
	glCullFace(GL_FRONT);
	glEnable(GL_TEXTURE_2D);

	glEnable(GL_ALPHA_TEST);
	glAlphaFunc(GL_GREATER, 0.666f);

	glPolygonMode (GL_FRONT_AND_BACK, GL_FILL);
	glShadeModel (GL_FLAT);

	glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
	glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
	glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
	glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);

	glBlendFunc (GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);

	glTexEnvf(GL_TEXTURE_ENV, GL_TEXTURE_ENV_MODE, GL_REPLACE);

}
```