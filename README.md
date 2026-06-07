# VaultAPI - Abstraction Library API for Bukkit Plugins

[![Maven Central](https://img.shields.io/maven-central/v/io.github.ulrichbr/VaultAPI?color=blue&label=Maven%20Central)](https://central.sonatype.com/artifact/io.github.ulrichbr/VaultAPI)
[![](https://jitpack.io/v/UlrichBR/VaultAPI.svg)](https://jitpack.io/#UlrichBR/VaultAPI)
[![Java Version](https://img.shields.io/badge/Java-8-orange?logo=openjdk)](https://pom.xml)

## 🚀 How to Integrate into Your Project

Choose one of the repositories below to add the VaultAPI API as a dependency in your build manager (Maven).

### Option 1: Maven Central (Recommended)
The official, fastest, and most stable method. It does not require adding any extra repository to your `pom.xml`, as the Maven ecosystem fetches the artifacts natively.

```xml
	implementation("io.github.ulrichbr:VaultAPI:$VERSION")
```

```xml
<dependency>
    <groupId>io.github.ulrichbr</groupId>
    <artifactId>VaultAPI</artifactId>
    <version>$VERSION</version>
</dependency>
```

### Option 2: JitPack (Alternative)
Use this option if you need to compile specific commits from branches or legacy versions hosted directly on the GitHub repository.

```xml

	dependencyResolutionManagement {
		repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
		repositories {
			mavenCentral()
			maven { url 'https://jitpack.io' }
		}
	}

	dependencies {
	        implementation 'com.github.UlrichBR:VaultAPI:$VERSION'
	}
```

```xml
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>[https://jitpack.io](https://jitpack.io)</url>
    </repository>
</repositories>

	<dependency>
	    <groupId>com.github.UlrichBR</groupId>
	    <artifactId>VaultAPI</artifactId>
	    <version>$VERSION</version>
	</dependency>
```


## Why Vault?
I have no preference which library suits your plugin and development efforts
best.  Really, I thought a central suite (rather...Vault) of solutions was the
the proper avenue than focusing on a single category of plugin.  That's where
the idea for Vault came into play.

So, what features do I _think_ you'll like the most?

 * No need to include my source code in your plugin
 * Broad range of supported plugins
 * Choice!

## License
Copyright (C) 2011-2018 Morgan Humes <morgan@lanaddict.com>

Vault is free software: you can redistribute it and/or modify
it under the terms of the GNU Lesser General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

Vault is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public License
along with Vault.  If not, see <http://www.gnu.org/licenses/>.

## Building
VaultAPI comes with all libraries needed to build from the current branch.

## Implementing Vault
Implementing Vault is quite simple. It requires getting the Economy, Permission, or Chat service from the Bukkit ServiceManager. See the example below:

```java
package com.example.plugin;

import net.milkbowl.vault.chat.Chat;
import net.milkbowl.vault.economy.Economy;
import net.milkbowl.vault.economy.EconomyResponse;
import net.milkbowl.vault.permission.Permission;

import org.bukkit.command.Command;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;
import org.bukkit.plugin.RegisteredServiceProvider;
import org.bukkit.plugin.java.JavaPlugin;

public class ExamplePlugin extends JavaPlugin {
    
    private static Economy econ = null;
    private static Permission perms = null;
    private static Chat chat = null;

    @Override
    public void onDisable() {
        getLogger().info(String.format("[%s] Disabled Version %s", getDescription().getName(), getDescription().getVersion()));
    }

    @Override
    public void onEnable() {
        if (!setupEconomy() ) {
            getLogger().severe(String.format("[%s] - Disabled due to no Vault dependency found!", getDescription().getName()));
            getServer().getPluginManager().disablePlugin(this);
            return;
        }
        setupPermissions();
        setupChat();
    }
    
    private boolean setupEconomy() {
        if (getServer().getPluginManager().getPlugin("Vault") == null) {
            return false;
        }
        RegisteredServiceProvider<Economy> rsp = getServer().getServicesManager().getRegistration(Economy.class);
        if (rsp == null) {
            return false;
        }
        econ = rsp.getProvider();
        return econ != null;
    }
    
    private boolean setupChat() {
        RegisteredServiceProvider<Chat> rsp = getServer().getServicesManager().getRegistration(Chat.class);
        chat = rsp.getProvider();
        return chat != null;
    }
    
    private boolean setupPermissions() {
        RegisteredServiceProvider<Permission> rsp = getServer().getServicesManager().getRegistration(Permission.class);
        perms = rsp.getProvider();
        return perms != null;
    }
    
    public boolean onCommand(CommandSender sender, Command command, String commandLabel, String[] args) {
        if(!(sender instanceof Player)) {
            getLogger().info("Only players are supported for this Example Plugin, but you should not do this!!!");
            return true;
        }
        
        Player player = (Player) sender;
        
        if(command.getLabel().equals("test-economy")) {
            // Lets give the player 1.05 currency (note that SOME economic plugins require rounding!)
            sender.sendMessage(String.format("You have %s", econ.format(econ.getBalance(player.getName()))));
            EconomyResponse r = econ.depositPlayer(player, 1.05);
            if(r.transactionSuccess()) {
                sender.sendMessage(String.format("You were given %s and now have %s", econ.format(r.amount), econ.format(r.balance)));
            } else {
                sender.sendMessage(String.format("An error occured: %s", r.errorMessage));
            }
            return true;
        } else if(command.getLabel().equals("test-permission")) {
            // Lets test if user has the node "example.plugin.awesome" to determine if they are awesome or just suck
            if(perms.has(player, "example.plugin.awesome")) {
                sender.sendMessage("You are awesome!");
            } else {
                sender.sendMessage("You suck!");
            }
            return true;
        } else {
            return false;
        }
    }
    
    public static Economy getEconomy() {
        return econ;
    }
    
    public static Permission getPermissions() {
        return perms;
    }
    
    public static Chat getChat() {
        return chat;
    }
}
```
