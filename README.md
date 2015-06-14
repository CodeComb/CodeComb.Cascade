在开发项目的过程中有如下的需求，即级联选择省、市、区县。

![QQ20150614-1@2x.png](http://www.1234.sh/file/download/557ce2233ec2882267dc2654)

由于select标签中的option不能隐藏，因此开发了此插件实现此类需求。

下面是示例代码，在html中引用codecomb.cascade.js，然后为select标签添加data-parent-select属性，指向另一个select的id，然后为option添加data-parent-id，绑定data-parent-select目标中的value。

**code** index.html

	<html>
	<head>
	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
		<script type="text/javascript" src="jquery-1.10.2.min.js"></script>
		<script type="text/javascript" src="codecomb.cascade.js"></script>
	</head>
	<body>
		<select id="province">
			<option value="1">黑龙江省</option>
			<option value="2">吉林省</option>
		</select>
		<select id="city" data-parent-select="province">
			<option value="1" data-parent-id="1">哈尔滨市</option>
			<option value="2" data-parent-id="1">齐齐哈尔市</option>
			<option value="3" data-parent-id="1">牡丹江市</option>
			<option value="4" data-parent-id="2">长春市</option>
		</select>
		<select id="town" data-parent-select="city">
			<option value="1" data-parent-id="1">南岗区</option>
			<option value="2" data-parent-id="1">道里区</option>
			<option value="3" data-parent-id="2">建华区</option>
			<option value="4" data-parent-id="2">龙沙区</option>
			<option value="5" data-parent-id="3">未知</option>
			<option value="6" data-parent-id="4">未知</option>
		</select>
	</body>
	</html>

最后放出插件源代码

**code** codecomb.cascade.js

	var __options = {};

	function setChildren(child)
	{
		if (child.length > 0)
		{
			for (var j = 0; j < child.length; j++)
			{
				var c = $(child[j]);
				c.html('');
				var id = c.attr('id');
				for (var k = 0; k < __options[id].length; k++)
				{
					if (!__options[id][k] || __options[id][k]['data-parent-id'] == $('#' + c.attr('data-parent-select')).val())
						c.append($('<option></option>').val(__options[id][k].value).text(__options[id][k].text));
				}
				setChildren($('select[data-parent-select="' + c.attr('id') + '"]'));
			}
		}
		else return;
	}

	$(document).ready(function () {
		var selects = $('select[data-parent-select]');
		for (var i = 0; i < selects.length; i++)
		{
			var dom = $(selects[i]);
			var id = dom.attr('data-parent-select');
			var srcdom = $('#' + id);
			if (__options[dom.attr('id')]) continue;
			__options[dom.attr('id')] = [];
			var options = dom.find('option');
			for (var j = 0; j < options.length; j++)
			{
				var opt = $(options[j]);
				__options[dom.attr('id')].push({ 
					'data-parent-id': opt.attr('data-parent-id'),
					'value': opt.attr('value'),
					'text': opt.text()
				});
			}
			srcdom.change(function() {
				$('select[data-parent-select="' + $(this).attr('id') + '"]').html('');
				var _id = $('select[data-parent-select="' + $(this).attr('id') + '"]').attr('id');
				var src = $(this);
				for (var j = 0; j < __options[_id].length; j++)
				{
					if (!__options[_id][j] || __options[_id][j]['data-parent-id'] == src.val())
					{
						$('select[data-parent-select="' + $(this).attr('id') + '"]').append('<option value="' + __options[_id][j].value + '">' + __options[_id][j].text + '</option>');
					}
				}
				var child = $('select[data-parent-select="' + $('select[data-parent-select="' + $(this).attr('id') + '"]').attr('id') + '"]');
				setChildren(child);
			});
		}
		for (var i = 0; i < selects.length; i++)
		{
			var dom = $(selects[i]);
			var id = dom.attr('data-parent-select');
			var srcdom = $('#' + id);
			srcdom.change();
		}
	});
