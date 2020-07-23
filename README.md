# Audio Occlusion in FiveM

This is an example set up for a 247 store. These files were exported from codewalker and then commented to create these instructions.

Importing these files in to the game will most likely not work, since the 247 store is already there.

You can reverse the 247 file contents and values using the below formulas to see and recognize patterns, and create your own custom interior audio files.

All Integers greater or lesser than `Int32.Max/Int32.Min` must be converted to `Int32`.

You can use the following: `$number & 0xffffffff`

Below are references to comments in the files and their purpose.

We've found ntOffset to not matter, set it to 0.

### References

    `stream/1690616543.ymt.pso.xml`

##  OcclusionHash

The formula to generate an occlusion hash is:

    `{$archetypeName.Hash} ^ Math.floor(({$mlo.pos.x} * 100)) ^ Math.floor(({$mlo.pos.y} * 100)) ^ Math.floor(({$mlo.pos.z} * 100)) & 0xffffffff`

##  FileName

File name is the Occlusion Hash

##  RoomIdx

This is the source room ID index as seen in the `<rooms>` key in the `ytyp` file.

##  DestRoomIdx

This is the destination room ID index as seen in the `<rooms>` key in the `ytyp` file.

##  PortalIdx

This is the portal ID based on the index of the `<portals>` key in the `ytyp` file.

Important note: this is the index of the `<portals>` list AFTER it has been filtered based on `RoomIdx`.

##  MaxOcclusion

Range from `0.0` to `1.0`.

##  hash_E3674005

Entity hash, of the door, window, etc.

##  PathNodeList

The PathNodeList is used describe audio paths between two rooms. That can be rooms directly connected such as `Room1 -> Portal0 -> Room2` (direct link) or rooms connected through multiple rooms and portals like `Room1 -> Portal0 -> Room2 -> Portal1 -> Room3` (bridge link). We'll call each entry in this list a `PathNode`.  

A "direct link" only links rooms together **explicitly**, meaning that even if you create `Room 0 -> Room 1` and `Room 1 -> 2`, audio is **not** routed from `Room 0 -> Room 2` as there is no direct link or a bridge link.

###  PathNodeList > Key

This key is unique per direction and is generated by using this formula: `(StartRoomHash - EndRoomHash)`.  

Below you can find a list of examples of key generation for  
`Room 0 -> Room 1`  
`Room 1 -> Room 0`  
`Room 1 -> Room 2`  
`Room 2 -> Room 1`  

-----------------------------------------------

**NOTE: WHEN GETTING JOAATS - ALWAYS USE LOWER CASE!**

-----------------------------------------------
##  PathNodeList > Key - Generation

GTA V uses 1-5 "channels" between each room, so the actual value you'll be using in the PathNodeList.Key is whatever the result of `(StartRoomHash - EndRoomHash)` is + `1 ... 5`. That means that you'll have to create `N` amount of PathNodes of each PathNodeList.Key. We usually just create three nodes. That means that for `Outside -> Room 1` which is `2124924645`, you'll have three PathNodes, with the `PathNodeList.Keys` [`2124924646`, `2124924647`, `2124924648`].

### Room definitions

| Name   | Ytyp Name    | joaat |
| :------------- | :----------: | -----------: |
| Outside | limbo   | `joaat("outside")` =  `3086856661` |
| Room 1   | V_66_ShopRm | `joaat("V_66_ShopRm")` = `1569794095` |
| Room 2   | V_66_BackRm | `joaat("V_66_BackRm")` = `462608346` |

>### **This is from the outside to the first room**
>
>  room1Hash = `occlusionHash ^ Room 1`
>
>            => 1690616543 ^ 1569794095
>
>            = **961932016**  
>
>  PathNodeList.Key = `Outside - room1Hash`
>
>            => 3086856661 - 961932016
>
>            = **2124924645**

>### **This is from the first room (the actual shop) to the outside**
> 
>     room1Hash = `occlusionHash ^ Room 1`
> 
>               => 1690616543 ^ 1569794095
> 
>               = **961932016**
> 
>     **PathNodeList.Key** = `room1Hash - Outside`
> 
>               => 961932016 - 3086856661
> 
>               = **-2124924645**

>### **This is from the first room to the back room**
> 
>     room1Hash = `occlusionHash ^ Room 2`
> 
>               => 1690616543 ^ 1569794095
> 
>               = **961932016**
> 
>     room2Hash = `occlusionHash ^ Room 2`
> 
>               => 1690616543 ^ 462608346
> 
>               = **2136347909**
> 
>     PathNodeList.Key = `room1Hash - room2Hash`
> 
>               => 961932016 - 2136347909
> 
>               = **-1174415893**

>### **This is from the back room to the first room**
> 
>     room2Hash = `occlusionHash ^ Room 2`
> 
>               => 1690616543 ^ 462608346
> 
>               = **2136347909**
> 
>     room1Hash = `occlusionHash ^ Room 1`
> 
>               => 1690616543 ^ 1569794095
> 
>               = **961932016**
> 
>     PathNodeList.Key = room2Hash - room1Hash
> 
>               => 2136347909 - 961932016
> 
>               = **1174415893**

When using and adding these keys, you will increment by `1` from the base value. They also need to be in an order where a key must be declared before it can be referenced in `PathNodeKey`.

##  PathNodeList > PathNodeChildList

As previously mentioned, a `PathNodeList.Key` must be unique Add `<Items>` in to this list to create multiple audio sources (in and out) for the same room.

##  PathNodeList > PathNodeChildList > PathNodeKey

If this value is `0`, its a direct reference to the `PortalInfoIdx` entry.

If this value is not `0`, it is a direct reference to another `PathNodeList.Key` value. This is used to link audio through multiple portals.

##   PathNodeList > PathNodeChildList > PortalInfoIdx

Reference to a `PortalInfoList` entry by index.

##  Garbage

Native GTA uses the same files for multiple interiors. You only want to reference the ones you are interested in.

##  Dat15

This is very straight forward.. 

File: audio/v_shop_247_mix.dat15.rel.xml

###  Scene

Set `<Name>$Dat15_Scene_Name</Name>` to whatever

Set `<Patch>$Dat15_Patch_Name</Patch>` to whatever

###  Patch

Set `<Name>$Dat15_Patch_Name</Name>` to the value you set in `Scene`.

In <Items> you can add definitions for ambient zone sound (limits or boosters(?)) - Check existing .dat15 for all the different variants you can use.

##  Dat151

###  AmbientZoneList <Item type=\"AmbientZoneList\"></Item>

Create an `<Item>$AmbientZoneName</Item>` with a name for the AmbientZone and add it to <Zones></Zones>

###  AmbientZone <Item type=\"AmbientZone\"></Item>

You can basically use most of whatever is in the template, but you'll have to change a few things.

Set OuterPos and InnerPos to the position the Ytyp and then you can use the following formula to calculate the size OuterSize (You should probably pad this with with 1.0) and InnerSize. Get the Minimum and Maximum Extents and use it to calculate the size of the box.

Set the name of the `<Name>$AmbientZoneName</Name>` to the name you decided in the AmbientZoneList($AmbientZoneName)

Set the name of `<UnkHash1>$Dat15_Scene_Name</UnkHash1>` to a name you set in Dat15

    
```
    Vector3 boxSize = extentsMax - extentsMin;

    Vector3 center = new Vector3(extentsMin.X + boxSize.X / 2, extentsMin.Y + boxSize.Y / 2, extentsMin.Z + boxSize.Z / 2)

    <OuterPos x="center.X" y="center.Y" z="center.Z" />

    <OuterSize x="boxSize.X" y="boxSize.Y" z="boxSize.Z" />

    <InnerPos x="center.X" y="center.Y" z="center.Z" />

    <InnerSize x="boxSize.X" y="boxSize.Y" z="boxSize.Z" />
```

Hashes contains the hashes of the staticEmitters - I wont go into detail about them.

###  Interior <Item type=\"Interior\"></Item>

Name needs to be the archetype (e.g. v_shop_247)

For each room in your interior you'll need to add an <Item>$WhateverYouWantAsANameForYourRoom</Item> in <Rooms></Rooms>, the name you choose however has to be the same name in "InteriorRoom".

###  InteriorRoom <Item type=\"InteriorRoom\"></Item>

File: audio/v_shop_247_game.dat151.rel.xml

For each room you have in your interior you have to create an entry with this (use this template basically and change anything with $)

 
```
      <Item type="InteriorRoom">

      <Name>$WhateverYouWantAsANameForYourRoom (from Interior)</Name>

      <Flags0 value="0xAAAAAAAA" />

      <MloRoom>$MloRoomName</MloRoom>

      <Hash1>$AmbientZone</Hash1>

      <Unk02 value="0" />

      <Unk03 value="0.5" />

      <Unk04 value="0" />

      <Unk05 value="0" />

      <Unk06>null_sound</Unk06>

      <Unk07 value="0" />

      <Unk08 value="0" />

      <Unk09 value="0" />

      <Unk10 value="0.55" />

      <Unk11 value="0" />

      <Unk12 value="50" />

      <Unk13 />

      <Unk14>hash_D4855127</Unk14>

    </Item>
```

You can tinker with the values, we haven't had the time to note down what they do exactly.

**Unk06** is the "room sound" or whatever (tone master?)

**Unk13** is the static emitter hash

## Credits (In no particular order)
Blade  
xIAlexanderIx  
Dwjft  
GottfriedLeibniz  
Everyone else that answered my questions :slight_smile:   
