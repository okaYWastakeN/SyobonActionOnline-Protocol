# Syobon Action Online Network Protocol Documentation

This repository contains documentation of the network protocol used by the Android version of the game "Syobon Action Online". This information was derived through reverse engineering the game's C# code (decompiled) and analyzing live UDP network traffic.

## Purpose

The primary goal of this documentation is to provide a clear understanding of how the Syobon Action Online client and server communicate, detailing the structure and purpose of various messages exchanged over UDP. This can be useful for:
* Educational purposes related to game networking and reverse engineering.
* Developing custom clients or server emulators (for personal use or community projects, respecting the original game creators).
* Deeper understanding of the game's internal mechanics.

## Protocol Overview

The game utilizes **User Datagram Protocol (UDP)** for its real-time network communications. Messages are generally structured with a 1-byte message type ID followed by variable payload data.

### Key Message Types (Client-to-Server)

| Type ID | Message Name | Purpose | Length | Structure | Notes |
| :------ | :----------- | :------ | :----- | :-------- | :---- |
| `0x03`  | `IAMHERE` (Player State Update) | Sent by the client to update its position, state, and appearance to the server. | 15 bytes | `[TypeID] [sta] [stb] [stc] [clientVersion] [mirror/visible_bits] [x (short)] [y (short)] [imgX] [imgY] [r] [g] [b]` | While the enum defines `IAMHERE` as `0x01`, observed traffic and client code show it's transmitted with `0x03` (which corresponds to `IAMME` in the enum). This suggests `0x03` is a general "my state" message. |

### Key Message Types (Server-to-Client)

| Type ID | Message Name | Purpose | Length | Structure | Notes |
| :------ | :----------- | :------ | :----- | :-------- | :---- |
| `0x02`  | `GAMELOOP` | Broadcasts the state of other players. | 1 byte or `1 + N * 13` bytes | `[TypeID]` followed by `N` x `SAClient` structures. | A 1-byte `0x02` indicates no other players, acting as a keep-alive. |
| `0x03`  | `SERVERSTATS` | Provides general server statistics (e.g., total online players). | 13 bytes | `[TypeID] [TotalOnline (int)] [TotalNearMe (int)] [ClientVersion (int)]` | This corresponds to `V2EXTRAS` in the server message enum, but the code explicitly uses it for `SERVERSTATS`. |
| `0x04`  | `WHOAREYOU` | Server challenges the client for its identity. | 1 byte | `[TypeID]` | Client responds with an `IAMME` (Type `0x03`) message containing Google Play Games display name and ID. |
| `0x05`  | `NEIGHBOR_NAMES` | Provides display names for nearby players. | Variable | `[TypeID]` followed by serialized `Neighbor` objects (including `r, g, b` and display name strings). | Used to display player names above character sprites. |

### `SAClient` Structure (13 bytes - used within `GAMELOOP` messages)

| Byte Offset(s) | Data Type | Field Name | Description |
| :------------- | :-------- | :--------- | :---------- |
| `0`            | `byte`    | `r`        | Red color component of player's sprite. |
| `1`            | `byte`    | `g`        | Green color component. |
| `2`            | `byte`    | `b`        | Blue color component. |
| `3`            | `byte`    | `sta`      | Player State A. |
| `4`            | `byte`    | `stb`      | Player State B. |
| `5`            | `byte`    | `stc`      | Player State C. |
| `6-7`          | `short`   | `x`        | X-coordinate of the player. |
| `8-9`          | `short`   | `y`        | Y-coordinate of the player. |
| `10`           | `byte`    | `imgX`     | Sprite sheet X-index. |
| `11`           | `byte`    | `imgY`     | Sprite sheet Y-index. |
| `12`           | `byte`    | `mirror` & `visible` | Bitfield: `(byte / 2 == 1)` for `mirror` (horizontal flip), `(byte % 2 == 1)` for `visible`. |

## Disclaimer

This documentation is based on analysis of the game's executable code and observed network traffic. It is unofficial and provided "as-is" for informational and educational purposes. The original game developers hold all rights to Syobon Action Online.
