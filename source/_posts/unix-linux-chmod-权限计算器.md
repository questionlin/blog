---
title: unix/linux chmod 权限计算器
date: 2018-11-07 11:37:12
tags: 在线工具
id: 1541565980
---
<form name="chmod">
<table border="0" align="center" cellpadding="4" cellspacing="0" class="datatable" style="font:normal 12px Verdana" ;="">
    <tbody><tr align="LEFT" valign="MIDDLE"> 
        <td class="adHeadline">Permissions:&nbsp; </td>
        <td>
        <input type="text" name="t_total" value="751" size="4" onkeyup="octalchange()">
        </td>
        <td>&nbsp; 
        <input type="text" name="sym_total" value="" size="12" readonly="1" style="border: 0px none; font-family: &quot;Courier New&quot;, Courier, mono;">
        </td>
    </tr>
    </tbody>
</table>
<br>
<table border="0" align="center" cellpadding="4" cellspacing="0" class="datatable" style="font:normal 12px Verdana">
    <tbody><tr bgcolor="#009900"> 
        <td width="60" align="left"> </td>
        <td width="55" align="center" style="color:white"><b>owner </b></td>
        <td width="55" align="center" style="color:white"><b>group </b></td>
        <td width="55" align="center" style="color:white"><b>other </b></td>
    </tr>
    <tr bgcolor="#dddddd"> 
        <td width="60" align="left" nowrap="" bgcolor="#FFFFFF">read</td>
        <td width="55" align="center" bgcolor="#EEEEEE"> 
        <input type="checkbox" name="owner4" value="4" onclick="calc_chmod()">
        </td>
        <td width="55" align="center" bgcolor="#ffffff">
        <input type="checkbox" name="group4" value="4" onclick="calc_chmod()">
        </td>
        <td width="55" align="center" bgcolor="#EEEEEE"> 
        <input type="checkbox" name="other4" value="4" onclick="calc_chmod()">
        </td>
    </tr>
    <tr bgcolor="#dddddd"> 
        <td width="60" align="left" nowrap="" bgcolor="#FFFFFF">write</td>
        <td width="55" align="center" bgcolor="#EEEEEE"> 
        <input type="checkbox" name="owner2" value="2" onclick="calc_chmod()">
        </td>
        <td width="55" align="center" bgcolor="#ffffff">
        <input type="checkbox" name="group2" value="2" onclick="calc_chmod()">
        </td>
        <td width="55" align="center" bgcolor="#EEEEEE"> 
        <input type="checkbox" name="other2" value="2" onclick="calc_chmod()">
        </td>
    </tr>
    <tr bgcolor="#dddddd"> 
        <td width="60" align="left" nowrap="" bgcolor="#FFFFFF">execute</td>
        <td width="55" align="center" bgcolor="#EEEEEE"> 
        <input type="checkbox" name="owner1" value="1" onclick="calc_chmod()">
        </td>
        <td width="55" align="center" bgcolor="#ffffff">
        <input type="checkbox" name="group1" value="1" onclick="calc_chmod()">
        </td>
        <td width="55" align="center" bgcolor="#EEEEEE"> 
        <input type="checkbox" name="other1" value="1" onclick="calc_chmod()">
        </td>
    </tr>
    </tbody>
</table>
</form>

<script>
function octalchange() 
{
	var val = document.chmod.t_total.value;
	var ownerbin = parseInt(val.charAt(0)).toString(2);
	while (ownerbin.length<3) { ownerbin="0"+ownerbin; };
	var groupbin = parseInt(val.charAt(1)).toString(2);
	while (groupbin.length<3) { groupbin="0"+groupbin; };
	var otherbin = parseInt(val.charAt(2)).toString(2);
	while (otherbin.length<3) { otherbin="0"+otherbin; };
	document.chmod.owner4.checked = parseInt(ownerbin.charAt(0)); 
	document.chmod.owner2.checked = parseInt(ownerbin.charAt(1));
	document.chmod.owner1.checked = parseInt(ownerbin.charAt(2));
	document.chmod.group4.checked = parseInt(groupbin.charAt(0)); 
	document.chmod.group2.checked = parseInt(groupbin.charAt(1));
	document.chmod.group1.checked = parseInt(groupbin.charAt(2));
	document.chmod.other4.checked = parseInt(otherbin.charAt(0)); 
	document.chmod.other2.checked = parseInt(otherbin.charAt(1));
	document.chmod.other1.checked = parseInt(otherbin.charAt(2));
	calc_chmod(1);
};

function calc_chmod(nototals)
{
  var users = new Array("owner", "group", "other");
  var totals = new Array("","","");
  var syms = new Array("","","");

	for (var i=0; i<users.length; i++)
	{
	  var user=users[i];
		var field4 = user + "4";
		var field2 = user + "2";
		var field1 = user + "1";
		//var total = "t_" + user;
		var symbolic = "sym_" + user;
		var number = 0;
		var sym_string = "";
	
		if (document.chmod[field4].checked == true) { number += 4; }
		if (document.chmod[field2].checked == true) { number += 2; }
		if (document.chmod[field1].checked == true) { number += 1; }
	
		if (document.chmod[field4].checked == true) {
			sym_string += "r";
		} else {
			sym_string += "-";
		}
		if (document.chmod[field2].checked == true) {
			sym_string += "w";
		} else {
			sym_string += "-";
		}
		if (document.chmod[field1].checked == true) {
			sym_string += "x";
		} else {
			sym_string += "-";
		}
	
		//if (number == 0) { number = ""; }
	  //document.chmod[total].value = 
		totals[i] = totals[i]+number;
		syms[i] =  syms[i]+sym_string;
	
  };
	if (!nototals) document.chmod.t_total.value = totals[0] + totals[1] + totals[2];
	document.chmod.sym_total.value = "-" + syms[0] + syms[1] + syms[2];
}
window.onload=octalchange
</script>

--------------------------------------
在UNIX系统家族里，文件或目录权限的控制分别以读取（Read)，写入(Write)，执行(Execute)3种一般权限来区分，另有3种特殊权限可供运用，再搭配拥有者与所属群组管理权限范围。您可以使用CHMOD指令去变更文件与目录的权限，设置方式采用文字或数字代号皆可。符号连接的权限无法变更，如果您对符号连接修改权限，其改变会作用在被连接的原始文件。权限范围的表示法如下：
- u：User，即文件或目录的拥有者。
- g：Group，即文件或目录的所属群组。
- o：Other，除了文件或目录拥有者或所属群组之外，其他用户皆属于这个范围。
- a：All，即全部的用户，包含拥有者，所属群组以及其他用户。
- 有关权限代号的部分，列表如下：
- r：读取权限，数字代号为“40”
- w：写入权限，数字代号为“2”
- x：执行或切换权限，数字代号为“10”
- -：不具任何权限，数字代号为“0”

--------------------------------------
转自：http://mistupid.com/internet/chmod.htm