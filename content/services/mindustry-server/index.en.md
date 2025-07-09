---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Gaming", "Strategies"]
description: "Server for industrial battles and tactical planning"
slug: "mindustry-server"
title: "Mindustry: Industrial Wars on Potato Energy"
---

### Real-time sandbox [🏭](https://mindustrygame.github.io/)

**Purpose** 
A strategic platform for:
- Building automated production chains
- Defending bases from waves of enemies
- Multiplayer PvP battles
- Creating custom scenarios

**Technical implementation**  
- Game version: **149**  
- Image: Official Docker image with GraalVM  
- Port: 6567 (TCP/UDP)  
- Storage: Local configs and maps  
- Performance: Optimized for ARM architectures  

**Security and Access**  
- Authentication: IP verification  
- Moderation: Built-in admin tools  
- Backup: Manual snapshots of the world  
- Networking: Isolated Docker segment  

**Features**  
- Support for 16 players simultaneously  
- Voting system for map changes  
- Synchronization with Discord game channel  
- Notifications about attack waves in Discord  
- Bot `mindustry#6994` for two-way communication

---

### Example server configuration
``properties
name=Potato Energy Battleground
map=fortress
mode=survival
difficulty=3
maxPlayers=16
````
## Settings are saved in the mounted volume ##

---

### Command system
```bash
# Start server with custom map
host --map nuclear-complex --mode attack
### Load custom mod
mod add "https://github.com/author/modname"
````

---

# Why is this important?
1. **Full control** over game balance  
2. **Learning about logistics** and process optimization  
3. **Development of teamwork** in PvE/PvP  
4. **Experiments** with production chains  

Join the industrial battles at `connect.potatoenergy.ru:6567`. Contact members of the `dev` group to access the admin panel.
