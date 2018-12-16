---
layout: post
title: "Reverse Engineering GeoMod Introduction"
date: 2018-12-16
---

## Reverse Engineering the GeoMod Engine Introduction

This is a little side project of mine.  The GeoMod engine was only used a few times, and only once to a degree of freedom that players both loved and hated.  Nobody really knows how it works, it’s a literal black box.  I’m going to start poking around the code with Radare2 and IDA to see what I can find out.  It should be fun.  But I think other interested people might want to go along for the ride so I decided to make a little blog about it.  It is worth mentioning that this code will be logically equivalent to the original GeoMod source, and the chances of a single person creating the precise code format and style as an entire team is highly unlikely.  And if I did have access to the source why would I bother going through all this trouble?  My sole reason for doing this project is to figure out what’s going on in that black box to pull out the engine and then update it to be useful in a modern application.
I’ll be reverse engineering Red Faction. First I’m looking in the actual game file, as opposed to the separate Launcher file. I figured that’s where they would put the game engine.
What I have found so far is that Red Faction is written in C++.  From the release date, it would have been written using C++99 standard.  The reason I know this is from syntax comments and libraries only available in C++. There are also references to .cpp files.  As a result I have to reverse the process by hand because most decompilers decompile to C and putting C++ into C is worse than manually converting it because there is too much missing information the decompiler needs and results in some really horrific code that is pretty much just assembly inside a C++ wrapper (which is as ugly as it sounds, so many void* and void**).  Now if you’re curious about IDA’s output, it’s not pretty at all.  There are “normal” functions, subroutines which use the call directive and are of the form sub_[hex line number]:.  Then there are the jump, loop and conditional branching locations marked as loc_[hex line number]:.  To say this is a mess would be understating it.  It’s more like what’s left after a bloody scene in a horror film and then recreating the scene from the aftermath.
However there is one function that shows up 135 times in the code.


`sub_4F9A80      proc near               ; CODE XREF: unknown_libname_1+5↑j`
`                                        ; unknown_libname_2+5↑j ...`
`                mov     eax, ecx ;arg_4 = return`
`                mov     dword ptr [eax], 0FFFFFFFFh ;; [eax] = 4294967295;`
`                retn ;return eax;`
`sub_4F9A80      endp`


Doesn’t seem like it does much, but the magical number eax is set to.  That just so happens to be 232-1 or U32MAX  but this can be shortened in C or C++ to:
`unsigned int result_to_arg_4(int *ret,  int *arg_4) { `
`	arg_4 =  ret;`
`	return UINT_MAX;`
`}`


Here’s a little gem from the code:
`.data:005A59EC                 db 'See John, or change MAX_FONTS in Graphics\Font.h',0Ah,0`

Now for the fun part seeing the GeoMod Engine in action.  
The developers included a special level called Glasshouse.  This is a sandbox level, no enemies, unlimited ammo for the rocket launcher, explosives, and assault rifle.  Though only the first two are good for really modifying geometry.  The assault rifle just breaks glass.  Speaking of which, there is a glass house in this room, but little else.  Time to play with the GeoMod engine: 

<video width="420" height="315">
<src="https://www.youtube.com/watch?v=1XHW4CPKL6Q">
</video>

