# vrp_rw_userlist

1. 해당 소스는 누구나 무료로 다운받아서 사용할 수 있습니다.

2. 해당 소스를 누구나 수정 또는 편집하여 무료로 재 배포할 수 있습니다.

3. 단, 해당 소스를 상업적인 목적으로 제3자에게 판매하는 행위를 금지합니다.

(*해당 소스가 정상적으로 작동하기 위해서는 맨 하단에 첨부한 vRP.getUserList() 가 필요합니다.)

- Fantastic design UI
- Scrollable list
- Unlimited number of users allowed
- Job tab
- Excellent server load protection
- Tested by over 250 people

© FiveM RealWorld. All rights reserved.

© FiveM RealWorld 모든 권리 보유.

<img src="https://github.com/fivem-realw/vrp_rw_userlist/blob/master/screenshots/logo.png?raw=true"></img>

- RealWorld Website: https://www.realw.kr
- RealWorld Discord: https://discord.gg/realw
- RealWorld Youtube: https://www.youtube.com/realw
- RealWorld Server: connect s.realw.kr

<img src="https://github.com/fivem-realw/vrp_rw_userlist/blob/master/screenshots/userlist.jpg?raw=true" width="100%"></img>

<h2>Function vRP.getUserList</h2>

(Please add to vrp/modules/identity.lua)

```lua

MySQL.createCommand("vRP/get_user_identities", "SELECT * FROM vrp_user_identities WHERE user_id in (@user_ids)")

function vRP.getUserIdentities(arr_user_ids, cbr)
  if cbr == nil then
    return
  end
  local task = Task(cbr)
  if arr_user_ids == nil or #arr_user_ids <= 0 then
    task({})
    return
  end
  MySQL.query(
    "vRP/get_user_identities",
    {user_ids = arr_user_ids},
    function(rows, affected)
      if rows == nil then
        task({})
      else
        task({rows})
      end
    end
  )
end

function GetJobType(user_id)
  local jobType = nil
  if vRP.hasPermission(user_id, "cop.whitelisted") then
    jobType = "cop"
  elseif vRP.hasPermission(user_id, "ems.whitelisted") then
    jobType = "ems"
  elseif vRP.hasPermission(user_id, "uber.whitelisted") then
    jobType = "uber"
  elseif vRP.hasPermission(user_id, "repair.whitelisted") then
    jobType = "repair"
  elseif vRP.hasPermission(user_id, "shh.whitelisted") then
    jobType = "shh"
  elseif vRP.hasPermission(user_id, "mafia.whitelisted") then
    jobType = "mafia"
  elseif vRP.hasPermission(user_id, "gm.whitelisted") then
    jobType = "gm"
  elseif vRP.hasPermission(user_id, "tow.whitelisted") then
    jobType = "tow"
  elseif vRP.hasPermission(user_id, "cbs.whitelisted") then
    jobType = "cbs"
  elseif vRP.hasPermission(user_id, "kys.whitelisted") then
    jobType = "kys"
  elseif vRP.hasPermission(user_id, "helper.whitelisted") then
    jobType = "helper"
  elseif vRP.hasPermission(user_id, "inspector.whitelisted") then
    jobType = "inspector"
  elseif vRP.hasPermission(user_id, "admin.whitelisted") then
    jobType = "admin"
  end
  return jobType
end

function GetUserTitleInfo(user_id)
  local tmpGroups = vRP.getUserGroups(user_id)
  local selUserTitle = nil
  for k, v in pairs(cfg_user_title.titles) do
    for k2, v2 in pairs(tmpGroups) do
      if k2 == v.group and v2 == true then
        selUserTitle = v
        break
      end
    end
    if selUserTitle ~= nil then
      break
    end
  end
  return selUserTitle
end

function GetArrUserGroups(user_id)
  local tmpGroups = vRP.getUserGroups(user_id)
  local arrGroups = {}
  for k2, v2 in pairs(tmpGroups) do
    if v2 == true then
      table.insert(arrGroups, k2)
    end
  end
  return arrGroups
end

local userList = {}
local user_ids = {}
local playerInfo = {}
function makeUserListTask()
  userList = {}
  user_ids = {}
  for _, v in ipairs(GetPlayers()) do
    playerInfo = {
      source = v,
      user_id = vRP.getUserId(v) or "",
      nickname = GetPlayerName(v) or "",
      name = "",
      job = "",
      jobType = ""
    }
    userList[playerInfo.user_id] = playerInfo
    table.insert(user_ids, playerInfo.user_id)
  end
  vRP.getUserIdentities(
    user_ids,
    function(identities)
      if identities == nil then
        SetTimeout(20000, makeUserListTask)
        return
      end
      for _, v in ipairs(identities) do
        if userList[v.user_id] then
          if v.firstname and v.name then
            userList[v.user_id].name = htmlEntities.encode(v.firstname) .. " " .. htmlEntities.encode(v.name)
          end
          userList[v.user_id].job = vRP.getUserGroupByType(userList[v.user_id].user_id, "job") or ""
          userList[v.user_id].jobType = GetJobType(userList[v.user_id].user_id) or ""
          userList[v.user_id].userTitleInfo = GetUserTitleInfo(v.user_id)
          userList[v.user_id].groups = GetArrUserGroups(v.user_id)
          userList[v.user_id].phone = v.phone
          if _ >= #identities then
            SetTimeout(20000, makeUserListTask)
          end
        end
      end
    end
  )
  user_ids = nil
end
makeUserListTask()

function vRP.getUserList(cbr)
  if cbr == nil then
    return
  end
  local task = Task(cbr)
  task({userList})
end
```

If you have any other questions, please contact fivemrealw@gmail.com
