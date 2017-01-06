# CD Music

Quake implements music by playing tracks off of the Quake CD while the game is running. 


## Overview 

The music system is defined in `cdaudio.h` and implemented in  `cd_sdl.c`. It suffers from a couple of limitations. First, there is no volume control from within Quake. It's either on or off. Second, not all CD-ROM drives report when they've finished playing a track, so there is no guarantee tracks will loop.

## Initialization

`CDAudio_Init`

## Playing a Track
When the server issues a play command, the client reacts by calling the `CDAudio_Play` function. (Called from `cl_parse.c:CL_ParseServerMessage`.)

`void CDAudio_Play(byte track, qboolean looping)`

**Params:** 

**track**    
The track number to play starting with 2. The 1st track on the Quake CD is reserved for the game data. Vanilla Quake has 10 tracks in total, numbered 2-11.

**looping**
True to loop the music, false to play once. Note that this does not work on all systems. 

_cd_sdl.c_
```
void CDAudio_Play(byte track, qboolean looping)
{
	int dwReturn;
	int trackLength;
	CDstatus status;
    
    /* Check to see that the CD player has been initialized and that there is a CD in the tray **/

	if (!enabled)
		return;
	
	if (!cdValid)
	{
		CDAudio_GetAudioDiskInfo();
		if (!cdValid)
			return;
	}

    /* Check for a valid track number */
	track = remap[track];

	status = SDL_CDStatus(cdinfo);

	if (track < 1 || track > maxTrack)
	{
		Con_DPrintf("CDAudio: Bad track number %u.\n", track);
		return;
	}
	
    /* Play the track */
    
	// don't try to play a non-audio track
	dwReturn = SDL_CDPlayTracks(cdinfo, track, 0, 0, cdinfo->track[track].length);
	if (dwReturn < 0)
	{
		Con_DPrintf("SDL_CDPlayTracks failed (%i)\n", dwReturn);
		return;
	}

	// get the length of the track to be played
	trackLength = cdinfo->track[track].length;
	
	playLooping = looping;
	playTrack = track;
	playing = true;

    /* Don't play if the volume is too low */
	if (cdvolume == 0.0)
		CDAudio_Pause ();
}
```


## Shutdown

