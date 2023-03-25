# qb-panicbutton

* Add a panic button to the pre-existing police job as an item inspired by nopixel

* Integrated with [ps-mdt]([https://github.com/Project-Sloth/ps-mdt](https://github.com/Project-Sloth/ps-dispatch))

* I will not be keeping up with or maintaining this 

* Support will be limited 

* Has support for ox resources (ox_inventory/ox_lib) and qb resources (qb-inventory/lj-inventory)

# Credit
Astro#3289 **for code and allowing me to release it**

# 
Muzzy#8363 **used some of his code and readme** (https://github.com/FiveOPZ/qb-panicbutton)

# Dependencies:

[qb-core](https://github.com/qbcore-framework/qb-core)

[qb-policejob](https://github.com/qbcore-framework/qb-policejob)

[ps-dispatch](https://github.com/Project-Sloth/ps-dispatch)

# Installation

# OX Installation

**goto ps-dispatch/client/cl_main.lua and add**

```lua
function IsDowned()
	return (PlayerData.metadata["isdead"] or PlayerData.metadata["inlaststand"])
end

local PressedPanicButton = false
RegisterNetEvent("ps-dispatch:client:TriggerPanicButton", function()
	if (PlayerData.job.name == "police" or PlayerData.job.type == "leo") or (PlayerData.job.name == "ambulance") then 
		if not PressedPanicButton then 
			QBCore.Functions.Progressbar("panic_button", "Pressing Panic Button...", 5000, false, true, {
				disableMovement = false,
				disableCarMovement = false,
				disableMouse = false,
				disableCombat = true,
			}, {}, {}, {}, function() 
				PressedPanicButton = true
				PlayerData = QBCore.Functions.GetPlayerData()
				if lib.callback.await("ps-dispatch:server:PressPanicButton") then 
					if (PlayerData.job.name == "police" or PlayerData.job.type == "leo") then
						TriggerServerEvent("InteractSound_SV:PlayWithinDistance", 7.0, "panicbutton", 7.0)
						exports['ps-dispatch']:OfficerDown2()
						PressedPanicButton = false
					elseif (PlayerData.job.name == "ambulance") then
						TriggerServerEvent("InteractSound_SV:PlayWithinDistance", 7.0, "panicbutton", 7.0)
						exports['ps-dispatch']:EMSDown2()
						PressedPanicButton = false
					end
				else
					PressedPanicButton = false
				end
			end, function()
				QBCore.Functions.Notify("Panic Button Canceled", "error")
			end)
		end
	end
end)

function UsedPanicButton()
	return PressedPanicButton
end exports("UsedPanicButton", UsedPanicButton)
```
**goto qb-ambulancejob/client/dead.lua and add**

```lua
function IsDowned()
	return (PlayerData.metadata["isdead"] or PlayerData.metadata["inlaststand"])
end

AddEventHandler('gameEventTriggered', function(event, data)
    if event == "CEventNetworkEntityDamage" then
        local victim, attacker, victimDied, weapon = data[1], data[2], data[4], data[7]
        if not IsEntityAPed(victim) then return end
        if victimDied and NetworkGetPlayerIndexFromPed(victim) == PlayerId() and IsEntityDead(PlayerPedId()) then
            if not InLaststand then
                SetLaststand(true)
				TimeToAlert = 30
				SentNotify = false
				PlayerData = QBCore.Functions.GetPlayerData()
				if not exports["ps-dispatch"]:UsedPanicButton() and (PlayerData.job.name == "police" or PlayerData.job.name == "ambulance") then 
					while true do 
						Wait(1000)
						PlayerData = QBCore.Functions.GetPlayerData()
						if IsDowned() then
							if TimeToAlert == 0 then return end
							if TimeToAlert ~= 0 then TimeToAlert = TimeToAlert - 1 end
							if lib.callback.await("ps-dispatch:server:PressPanicButton") then 
								if not SentNotify then QBCore.Functions.Notify("Vital Signs Low, Sending Alert In "..TimeToAlert.." Second(s)") SentNotify = true end
								if TimeToAlert == 0 then 
									if (PlayerData.job.name == "police" or PlayerData.job.type == "leo") then
										TriggerServerEvent("InteractSound_SV:PlayWithinDistance", 7.0, "panicbutton", 7.0)
										exports['ps-dispatch']:OfficerDown()
										break
									elseif (PlayerData.job.name == "ambulance") then
										TriggerServerEvent("InteractSound_SV:PlayWithinDistance", 7.0, "panicbutton", 7.0)
										exports['ps-dispatch']:EMSDown()
										break
									end
								end
							else
								break
							end
						else 
							break
						end
					end
				end
			end
            elseif InLaststand and not isDead then
                SetLaststand(false)
                local playerid = NetworkGetPlayerIndexFromPed(victim)
                local playerName = GetPlayerName(playerid) .. " " .. "("..GetPlayerServerId(playerid)..")" or Lang:t('info.self_death')
                local killerId = NetworkGetPlayerIndexFromPed(attacker)
                local killerName = GetPlayerName(killerId) .. " " .. "("..GetPlayerServerId(killerId)..")" or Lang:t('info.self_death')
                local weaponLabel = QBCore.Shared.Weapons[weapon].label or 'Unknown'
                local weaponName = QBCore.Shared.Weapons[weapon].name or 'Unknown'
                TriggerServerEvent("qb-log:server:CreateLog", "death", Lang:t('logs.death_log_title', {playername = playerName, playerid = GetPlayerServerId(playerid)}), "red", Lang:t('logs.death_log_message', {killername = killerName, playername = playerName, weaponlabel = weaponLabel, weaponname = weaponName}))
                deathTime = Config.DeathTime
                OnDeath()
                DeathTimer()
            end
        end
    end
end)
```
**goto qb-ps-dispatch/server/sv_main.lua and add**
```lua
QBCore.Functions.CreateUseableItem("panicbutton", function(source, item)
	TriggerClientEvent("ps-dispatch:client:TriggerPanicButton", source)
end)

lib.callback.register("ps-dispatch:server:PressPanicButton", function()
	if next(exports.ox_inventory:Search(source, "slots", "panicbutton")) ~= nil then 
		return true
	else
		return false
	end
end)
```

# QB Installation

**goto ps-dispatch/client/cl_main.lua and add**

```lua
function IsDowned()
	return (PlayerData.metadata["isdead"] or PlayerData.metadata["inlaststand"])
end

local PressedPanicButton = false
RegisterNetEvent("ps-dispatch:client:TriggerPanicButton", function()
	if (PlayerData.job.name == "police" or PlayerData.job.type == "leo") or (PlayerData.job.name == "ambulance") then 
		if not PressedPanicButton then 
			QBCore.Functions.Progressbar("panic_button", "Pressing Panic Button...", 5000, false, true, {
				disableMovement = false,
				disableCarMovement = false,
				disableMouse = false,
				disableCombat = true,
			}, {}, {}, {}, function() 
				PressedPanicButton = true
				PlayerData = QBCore.Functions.GetPlayerData()
				QBCore.Functions.TriggerCallback("ps-dispatch:server:PressPanicButton", function(HasItem)
					if HasItem then 
						if (PlayerData.job.name == "police" or PlayerData.job.type == "leo") then
							TriggerServerEvent("InteractSound_SV:PlayWithinDistance", 7.0, "panicbutton", 7.0)
							exports['ps-dispatch']:OfficerDown2()
							PressedPanicButton = false
						elseif (PlayerData.job.name == "ambulance") then
							TriggerServerEvent("InteractSound_SV:PlayWithinDistance", 7.0, "panicbutton", 7.0)
							exports['ps-dispatch']:EMSDown2()
							PressedPanicButton = false
						end
					else
						PressedPanicButton = false
					end
				end)
			end, function()
				QBCore.Functions.Notify("Panic Button Canceled", "error")
			end)
		end
	end
end)

function UsedPanicButton()
	return PressedPanicButton
end exports("UsedPanicButton", UsedPanicButton)
```
**goto qb-ambulancejob/client/dead.lua and add**

```lua
function IsDowned()
	return (PlayerData.metadata["isdead"] or PlayerData.metadata["inlaststand"])
end

AddEventHandler('gameEventTriggered', function(event, data)
    if event == "CEventNetworkEntityDamage" then
        local victim, attacker, victimDied, weapon = data[1], data[2], data[4], data[7]
        if not IsEntityAPed(victim) then return end
        if victimDied and NetworkGetPlayerIndexFromPed(victim) == PlayerId() and IsEntityDead(PlayerPedId()) then
            if not InLaststand then
                SetLaststand(true)
				TimeToAlert = 30
				SentNotify = false
				SentAlert = false
				PlayerData = QBCore.Functions.GetPlayerData()
				if not exports["ps-dispatch"]:UsedPanicButton() and (PlayerData.job.name == "police" or PlayerData.job.name == "ambulance") then 
				while true do 
					Wait(1000)
					PlayerData = QBCore.Functions.GetPlayerData()
					if IsDowned() then
						if TimeToAlert == 0 then return end
						if TimeToAlert ~= 0 then TimeToAlert = TimeToAlert - 1 end
						if SentAlert then break end
						QBCore.Functions.TriggerCallback("ps-dispatch:server:PressPanicButton", function(HasItem)
							if HasItem then 
								if not SentNotify then QBCore.Functions.Notify("Vital Signs Low, Sending Alert In "..TimeToAlert.." Second(s)") SentNotify = true end
								if TimeToAlert == 0 then 
									if (PlayerData.job.name == "police" or PlayerData.job.type == "leo") then
										TriggerServerEvent("InteractSound_SV:PlayWithinDistance", 7.0, "panicbutton", 7.0)
										exports['ps-dispatch']:OfficerDown()
										SentAlert = true
									elseif (PlayerData.job.name == "ambulance") then
										TriggerServerEvent("InteractSound_SV:PlayWithinDistance", 7.0, "panicbutton", 7.0)
										exports['ps-dispatch']:EMSDown()
										SentAlert = true
									end
								end
							end
						end)
					else 
						break
					end
					if SentAlert then 
						break
					end
				end
			end
            elseif InLaststand and not isDead then
                SetLaststand(false)
                local playerid = NetworkGetPlayerIndexFromPed(victim)
                local playerName = GetPlayerName(playerid) .. " " .. "("..GetPlayerServerId(playerid)..")" or Lang:t('info.self_death')
                local killerId = NetworkGetPlayerIndexFromPed(attacker)
                local killerName = GetPlayerName(killerId) .. " " .. "("..GetPlayerServerId(killerId)..")" or Lang:t('info.self_death')
                local weaponLabel = QBCore.Shared.Weapons[weapon].label or 'Unknown'
                local weaponName = QBCore.Shared.Weapons[weapon].name or 'Unknown'
                TriggerServerEvent("qb-log:server:CreateLog", "death", Lang:t('logs.death_log_title', {playername = playerName, playerid = GetPlayerServerId(playerid)}), "red", Lang:t('logs.death_log_message', {killername = killerName, playername = playerName, weaponlabel = weaponLabel, weaponname = weaponName}))
                deathTime = Config.DeathTime
                OnDeath()
                DeathTimer()
            end
        end
    end
end)
```
**goto qb-ps-dispatch/server/sv_main.lua and add**
```lua
QBCore.Functions.CreateUseableItem("panicbutton", function(source, item)
	TriggerClientEvent("ps-dispatch:client:TriggerPanicButton", source)
end)

QBCore.Functions.CreateCallback("ps-dispatch:server:PressPanicButton", function(source, cb)
	local Player = QBCore.Functions.GetPlayer(source)
	if Player.Functions.GetItemByName("panicbutton") then 
		cb(true)
	else
		cb(false)
	end
end)
```


# Alert Installation
**goto qb-core/shared/items.lua and add**

```lua
['panicbutton'] = {['name'] = 'panicbutton',['label'] = 'Panic Button',["created"] = nil, ['weight'] = 100,	['type']= 'item',['image'] = 'panicbutton.png', ['unique'] = true,	['useable']= true ['shouldClose'] =true,['combinable'] = nil,['description'] = 'Some sort of button?'},
```
**ps-dispatch/server/sv_dispatchcodes.lua**
**find ["officerdown"] and replace with**

```lua
["officerdown"] =  {displayCode = '10-13a', description = "Officer Down", radius = 15.0, recipientList = {'police'}, blipSprite = 526, blipColour = 1, blipScale = 1.5, blipLength = 2, sound = "panicbutton", offset = "false", blipflash = "false"},
["officerdown2"] =  {displayCode = '10-78', description = "Officer Needs Assistance", radius = 15.0, recipientList = {'police'}, blipSprite = 526, blipColour = 1, blipScale = 1.5, blipLength = 2, sound = "panicbutton", offset = "false", blipflash = "false"},
["emsdown"] =  {displayCode = '10-13a', description = "EMS Down", radius = 15.0, recipientList = {'police'}, blipSprite = 526, blipColour = 1, blipScale = 1.5, blipLength = 2, sound = "panicbutton", offset = "false", blipflash = "false"},
["emsdown2"] =  {displayCode = '10-78', description = "EMS Needs Assistance", radius = 15.0, recipientList = {'police'}, blipSprite = 526, blipColour = 1, blipScale = 1.5, blipLength = 2, sound = "panicbutton", offset = "false", blipflash = "false"},
```
**ps-dispatch/locales/locales.lua**
**find ['officerdown'] and replace with**
```lua
['officerdown'] = "Officer Down",
['officerdown2'] = "Officer Needs Assistance",
['emsdown'] = "EMS Down",
['emsdown2'] = "EMS Needs Assistance",
```
**ps-dispatch/client/cl_events.lua**
**Find** 

```lua
local function OfficerDown()
```
**and**

```lua
local function EmsDown()
```
**and replace with**

```lua
local function OfficerDown()
    local plyData = QBCore.Functions.GetPlayerData()
    local currentPos = GetEntityCoords(PlayerPedId())
    local locationInfo = getStreetandZone(currentPos)
    local callsign = QBCore.Functions.GetPlayerData().metadata["callsign"]
    TriggerServerEvent("dispatch:server:notify", {
        dispatchcodename = "officerdown", -- has to match the codes in sv_dispatchcodes.lua so that it generates the right blip
        dispatchCode = "10-13A",
        firstStreet = locationInfo,
        model = nil,
        plate = nil,
        callsign = callsign,
        priority = 2, -- priority
        firstColor = nil,
        automaticGunfire = false,
        origin = {
            x = currentPos.x,
            y = currentPos.y,
            z = currentPos.z
        },
        dispatchMessage = _U('officerdown'), -- message
        job = { "ambulance", "police" } -- jobs that will get the alerts
    })
end

exports('OfficerDown', OfficerDown)

local function OfficerDown2()
    local plyData = QBCore.Functions.GetPlayerData()
    local currentPos = GetEntityCoords(PlayerPedId())
    local locationInfo = getStreetandZone(currentPos)
    local callsign = QBCore.Functions.GetPlayerData().metadata["callsign"]
    TriggerServerEvent("dispatch:server:notify", {
        dispatchcodename = "officerdown2", -- has to match the codes in sv_dispatchcodes.lua so that it generates the right blip
        dispatchCode = "10-78",
        firstStreet = locationInfo,
        model = nil,
        plate = nil,
        callsign = callsign,
        priority = 2, -- priority
        firstColor = nil,
        automaticGunfire = false,
        origin = {
            x = currentPos.x,
            y = currentPos.y,
            z = currentPos.z
        },
        dispatchMessage = _U('officerdown2'), -- message
        job = { "ambulance", "police" } -- jobs that will get the alerts
    })
end

exports('OfficerDown2', OfficerDown2)

local function EmsDown()
    local plyData = QBCore.Functions.GetPlayerData()
    local currentPos = GetEntityCoords(PlayerPedId())
    local locationInfo = getStreetandZone(currentPos)
    local callsign = QBCore.Functions.GetPlayerData().metadata["callsign"]
    TriggerServerEvent("dispatch:server:notify", {
        dispatchcodename = "emsdown", -- has to match the codes in sv_dispatchcodes.lua so that it generates the right blip
        dispatchCode = "10-13A",
        firstStreet = locationInfo,
        model = nil,
        plate = nil,
        callsign = callsign,
        priority = 2, -- priority
        firstColor = nil,
        automaticGunfire = false,
        origin = {
            x = currentPos.x,
            y = currentPos.y,
            z = currentPos.z
        },
        dispatchMessage = _U('emsdown'), -- message
        job = { "ambulance", "police" } -- jobs that will get the alerts
    })
end

exports('EmsDown', EmsDown)

local function EmsDown2()
    local plyData = QBCore.Functions.GetPlayerData()
    local currentPos = GetEntityCoords(PlayerPedId())
    local locationInfo = getStreetandZone(currentPos)
    local callsign = QBCore.Functions.GetPlayerData().metadata["callsign"]
    TriggerServerEvent("dispatch:server:notify", {
        dispatchcodename = "emsdown2", -- has to match the codes in sv_dispatchcodes.lua so that it generates the right blip
        dispatchCode = "10-78",
        firstStreet = locationInfo,
        model = nil,
        plate = nil,
        callsign = callsign,
        priority = 2, -- priority
        firstColor = nil,
        automaticGunfire = false,
        origin = {
            x = currentPos.x,
            y = currentPos.y,
            z = currentPos.z
        },
        dispatchMessage = _U('emsdown2'), -- message
        job = { "ambulance", "police" } -- jobs that will get the alerts
    })
end

exports('EmsDown2', EmsDown2)
```
