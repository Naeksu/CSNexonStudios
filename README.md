![alt text](https://i.imgur.com/m11SthZ.png)
# Remark of 2022
This repo contains a compilation I’ve made over the past years. I would like to completely remake everything, but, alas, I’m too late. The game is dead and I don’t feel like loosing more time working on a dead project. 
The remainder of this document is left as is.
___
# Content
## Dependency Graph
![alt text](https://i.imgur.com/DXgLVoR.png)

Third-Party Data
---
**Hallf-Life SDK**

SDK for the GoldSource engine games. Originally intended for HL1, CS 1.6 and associated games. In fact CSN:S is based on CS:CZ engine (modified version of GoldSource), also most of SDK structures was modified by Nexon, but this SDK still may be used with CSN:S. SDK contained in this solution is partially reconstructed (using reverse-engineering) and ready to use with CSN:S. Authors of reconstruction: Shatilova, h4rdee, JusicP & Naeksu.

**Changelog:**

- Duplicate files inside different directories deleted.
- New functions inside efx_api_s and IPanel found. Their purpose is unknown.
- New fields found inside hud_player_info_s, cl_entity_s, clientdata_s, playermove_s, weapon_data_s. Purpose of most of them is unknown.
- cl_enginefuncs_s is partially recontructed:
- There are a lot of new functions that return pointers to new ingame classes instances. Some of these functions are named, some are not.
- There are client-table functions callbacks. Unlike the corresponding client-table function, callback doesn't take any arguments, and always return 'int'. I've no idea about initial purpose of these callbacks, but probably it was a part of client-table functions protection.
- enginefuncs_s is partially reconstructed: it also contains many new functions that return pointers to new ingame classes instances. Some of these functions are named, some are not.
- _studio_interface_s contains two new funtions: 
- StudioDrawShopModel - invoked when player is drawn in Inventory/Game Shop tabs. I remember I found there is some mutable data on function stack that can be modified to change player costumes (visual only, of course), but I lost any information about it.
- Unknown function that is invoked every game frame. Should be researched. Also we can obtain pointer to 'StudioModelRenderer' instance by offset of that function: (DWORD)((DWORD)((DWORD)r_studio_interface_s::Unknown + 0x1))
- Copy-constructor and copy-assignment operator are defined for the following structures: cl_enginefuncs_s, engine_studio_api_s, r_studio_interface_s, cl_clientfuncs_s.
- Added two structures: client_state_s and client_static_s. Both of them have undergone major changes. client_state_s is still changed while big game updates. I wasn't interested in reconstructing it, so I just hidden unknown areas under comments.
- entity_state_s has weaponpainttype field that stores information about paint type of current weapon.

The following structures most certainly haven't changed: con_nprint_s, cvar_s, dlight_t, event_hook_t, screenfade_s, SCREENINFO_s, xcommand_t, local_state_s, engine_studio_api_s, TUserMsg, IGameConsole, kbutton_s.

**GLEW**

Cross-Platform, Open Source C/C++ extension loading lib.

**GLFW**

Open Source, Multi-Platform lib for OpenGL, OpenGL ES & Vulkan application development.


**ImGui**

Bloat-free Graphical UI for C++ with minimal dependencies. The lib was supplemented with a new widget - Hotkey. Widget Code Author: Naeksu

**Anti Reverse-Engineering Module**

Set of techinques to prevent debugging and reverse-engineering the application. OG Author: Joshua Jackson | RE-Modded by: Naeksu

**VMT Hook**

VMT Hook code, OG Author: Naeksu

___

# Tools

---

## Production Lib's

**Easy Packer**

Cross-Platform Lib to process files & compile them into binary files and vice versa. Features:

- Glue files into binary file
- Load files to memory as RAW bytes (include basic file information like filename & size)
- Unpack files
- Unpack files to memory as raw bytes (include basic file information like filename & size)

Allows you to change magic number & extension of binary file.

**Easy HWID**

Lib to get unique ID. All techniques are taken from the web. Using all of the included methods can provide strong HWID protection.

**SoftON Socket**

Specific Lib that implements socket-client with simple encryption of transferred data. You can read more about client-server relationship below.

___

# Main Projects


**File Packer**

Console application to pack fines into *.dat binary files. Application must be placed in directory with files to pack.
![alt text](https://i.imgur.com/cm6xLBJ.png)

**DLL**

The **software**

![alt text](https://i.imgur.com/VDPkjYB.png)
![alt text](https://i.imgur.com/xalgPkT.png)
![alt text](https://i.imgur.com/3rR1sml.png)

**Loader**

The cheat loader that receives data files contain images and font used by loader & dll, then recieves DLL from the server, then injects it.

![alt text](https://i.imgur.com/lkCVTUm.png)
___

# Client-Server Relationship

![alt text](https://i.imgur.com/SXTP41c.png)

Firstly; Client-Server uses two methods to protect hte transferred data:

### Encryption

There are two, 20 character long keys: Static & Dynamic

- Static: Is known to both client & server. It's used by the client to encrypt data already encrypted with dynamic keys, then used by the server to decrypt double-encrypted data.
- Dynamic: Is generated by the client everytime it sends data to the server. That key is sent to the server with the encrypted data.

### Shuffling

The server generates a key thats used for a reversible shuffle algorithm. That key is sent to the client in order to shuffle data back to the server.
___


### Stages

Sending data to the server (data sent to the server is always encrypted, not shuffled)

1. Dynamic key is generated
2. Data to transfer is encrypted with dynamic key
3. Encrypted data is concatenated with dynamic key
4. The resulting data is encrypted with static key
5. Double-encrypted data is sent to the server

### Receiving data from the client

1. Received double-encrypted data is decrypted with static key
2. Since key length is static, dynamic key is just seperated from the data
3. Encrypted with dynamic key data is decrypted with dynamic key
4. Decrypted data is processed

**Sending data to the client (IE: Shuffled Data)

1. The random key for shuffling is generated
2. Header of transferring data is built
3. Header is encrypted with dynamic key (that was received from the client)
4. Header is sent to the client
5. The data to send is shuffled with generated random key
6. The data is sent to the client

**Receiving data from the server(IE: Shuffled Data)

1. The data header is received and decrypted with a dynamic key
2. The random key is extracted from the decrypted header
3. The data is received & deshuffled with a random key
4. The data ir processed.
___

# Project Structure

### Client
- Anti-RE, Easy-HWID, Easy-Packer, SoftON-Socket - Library projects that are used by main projects.
- DLL, File-Packer, Loader - Main projects
- Assets - Unpacked files are used by loader/dll
- Bin - built main projects
- Build - Object files
- Lib - Built libraries

### Images - Directory contains images that's used by this README
### Server
- *.py - Server source code
- DLL.DLL - The cheat itself
- SoftON_Into.txt - Text htat displayed in a "Hack Info" field of the loader
- SoftON_cmd_history.log - Log contains information about any & all changes of the users dB
- Loader.dat - Data-file with files used by loader
- SoftON.dat - Data-file with files used by dll
- Users_Info.SQLITE - Users dB
___

# Usage Guide

**Using Local Server**

- Build the solution using MSVS19 OR higher (C++ 17 Standard is minimum **req**)
- Create data-files for loader and DLL files using **file-packer.exe** from **client/bin/** (file names are *Loader* / *SoftON*)
- Place created data-files to ***Server/***
- Place **DLL.DLL** from **client/bin/** to **server/**
- Run **server.py** from **server/**
- Run the game
- Run the **loader.exe** from **client/bin/**
- Press *Inject* **ONCE**, Check console for a request message
- Add yourself to the database using the ***add** command
- Press *Inject* once more

**Using VPS**

- Change *IP* & *Port* constants inside **main.cpp* of **loader** project and **main.cpp** of **DLL** project to **IP** & **Por** of your **VPS**
- Build the solution using MSVS19 or higher (C++ 17 Standard is minimum **req**)
- Change *IP* & *PORT* constants inside **server.py** from **server/** to **IP** % **PORT** of your **VPS**
- Place everything from **server/** to your **VPS**
- Create data-files for loader and DLL using **file.packer.exe** from **client/bin/** (file names are *Loader* & *SoftON*)
- Place created data-files in your **VPS**
- Place **DLL.DLL** from **client/bin/** to your **VPS**
- Run **server.py** from your **VPS**
- Run the game
- Run the **Loader.exe** from **client/bin/**
- Press *Inject* **ONCE**, Check console for a request message
- Add yourself to the database using the **add** command
- Press *Inject" once more

**Without Any Server**

- Comment *AuthServer* function to invoke inside **main.cpp** of **DLL** project
- Build the solution using MSVS19 or higher (C++ 17 Standard is minimum **req**)
- Create data-file for the DLL using **file-packer.exe** from **client/bin/** (Filename: SoftON)
- Place created data-file to ...**/AppData/Roaming/SoftON** (Create "**SoftON**" folder if non-existing
- Run the game
- Run any injector you would like, inject **DLL.DLL* from **client/bin/**
