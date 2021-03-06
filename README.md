![](https://i.imgur.com/ikil8Xm.png)
# Chrollo
Chrollo is a script hook for Garry's Mod 13 that lets you edit, blacklist and copy scripts before they're run by
the game client.

A common use for Chrollo is to bypass anti-cheats on servers which rely entirely on Lua code execution on the clientside. You can see **every** script your client runs. 

Chrollo also serves as a script enforcer bypass as well.

# Features

Currently with Chrollo you can

- Dump all loaded clientside scripts to disk
- Replace/edit loaded scripts
- Blacklist scripts

# Installation

1. Download the latest release from the [releases](https://github.com/qubard/GMOD-Script-Leaker/releases) page
2. Open Garry's Mod
3. Inject the `.dll` file into your game.
4. Load a multiplayer server.
5. All downloaded scripts can be found in the `steamapps/common/GarrysMod/chrollo/<server name>/` directory in your `Garry's Mod` folder.

# Updating

Upon each game version change the only information that needs to be updated is the `server_name` pointer. I'll try to keep each version updated, but it's easy to find the server name pointer with something like CheatEngine (search for the server name as a string in the binary--it's always a static offset).

# Replacing scripts

In `chrollo/replace` save your edited script with the same name (delimited by `_`) as the script you want to edit. That's it, and when you inject Chrollo your script will be replaced.

# Blacklisting scripts 

In `chrollo/blacklist.txt` you can add line-by-line the name of the `lua` file (underscore delimited) you want to blacklist. Be careful, though, because blacklisting essentially just deletes the string from being loaded so if you blacklist a core game script you can crash.

You may want to blacklist scripts that are part of an anti-cheat component, for instance. Blacklisted scripts do not run.

Sample `blacklist.txt` file to bypass most TTT anti-cheats

```
addons_moat_addons_lua_plugins_moat_modules_snap_utils.lua
addons_moat_addons_lua_plugins_moat_modules_snap_init.lua
addons_moat_addons_lua_plugins_moat_modules_snap_client_menu.lua
addons_moat_addons_lua_plugins_moat_modules_snap_client_client.lua
addons_1_sgm_lua_gcap_client_gcap_client.lua
gamemodes_sandbox_entities_effects_propspawn.lua
addons_leyscreencap_lua_autorun_client_cl_leyscreencap.lua
addons_swiftac_lua_swiftac.lua
addons_ulx_lua_ulx_modules_sh_cc_virus.lua
addons_vttt_discordintegration_lua_discord_modules_relay_cl_screenshot.lua
addons_pac3-master_lua_pac3_core_client_parts_censor.lua
addons_snapper_lua_screensnapper_client.lua
addons_crescent_lua_crescent_modules_snap_cl_snap.lua
```

# Why use this?

You can find a lot of cool stuff with this. I was able to bypass all of [moat.gg](https://moat.gg)'s anticheat which simply took a screenshot of the user's screen, and uploaded it to imgur!

```lua
...
net.Receive("Snapper Victim", function(length, server)
    local ply = net.ReadEntity()
    if IsValid(ply) and ply:IsPlayer() then
        snapper.victim = ply
    end
end)

net.Receive("Snapper Notify", function(length, server)
	local contents = net.ReadTable()

	if not contents then
		return
	end

	chat.AddText(Material("icon16/information.png"), unpack(contents))
end)

local function mupload()
    local a = math.Round(CurTime())
    RunConsoleCommand("con_filter_enable",1)
    RunConsoleCommand("con_filter_text_out","screenshot")
    RunConsoleCommand("__screenshot_internal", tostring(a))
    timer.Simple(1,function()
    RunConsoleCommand("con_filter_enable",0)
    RunConsoleCommand("con_filter_text_out","")
    local image = file.Read("screenshots/" .. tostring(a) .. ".jpg","GAME")
        HTTP({
            url = "https://api.imgur.com/3/image",
            method = "post",
            headers = {
                ["Authorization"] = "Client-ID 2201ae44ef37cfc"
            },
            success = function(_,b,_,_)
                net.Start("moat-ab")
                net.WriteBool(true)
                net.WriteString(b)
                net.SendToServer()
            end,
            failed = function(b) 
                net.Start("moat-ab")
                net.WriteBool(false)
                net.WriteString(b)
                net.SendToServer()
            end,
            parameters = {
                image = util.Base64Encode(image)
            },
        })
    end)
end
net.Receive("moat-ab",mupload)
...
```