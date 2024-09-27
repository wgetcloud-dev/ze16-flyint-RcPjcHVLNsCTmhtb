
之前的文章说了，使用反射和ABPvNext的Dto实现用后端控制前端以实现最佳CRUD实践！
相信看过一的已经了解了这个PasteForm是如何实现的了，本文来看下具体如何实现的


## 表格页面的实现


打开pasteform/index.html页面之后，先会向API请求当前的path的数据模板



```
    _apiget(`/api/app/${_classPath}/readListModel`, true, (c, o) => {
        if (c == 200) {
            if (o.title) {
                $(".ppbody .st").find(".sn").html(o.title);
                this.document.title = o.title;
            }
            if (o.desc) {
                $(".ppbody .st").find(".idesc").html(o.desc);
            }
            //表头的模板内容
            var _template_head_html = null;

            _globadataProperties = o.properties;
            //模型处理，如何显示外表 比如cate.name 
            HandlerModelColumn(o.properties);
            //class模型的属性列表
            if (o.attributes) {
                _globadataAttributes = o.attributes;
                o.attributes.forEach(_attribute => {
                    if (_attribute.name == 'disable') {
                        if (_attribute.args1) {
                            _config.disable_add = true;
                            $(".btnadd").hide();
                        }
                        if (_attribute.args2) {
                            _config.disable_edit = true;
                        }
                        if (_attribute.args3) {
                            _config.disable_del = true;
                        }
                    }
                    if (_attribute.name == "template") {
                        if (_attribute.args1) {
                            _template_head_html = $(`#${_attribute.args1}`).html();
                        }
                        if (_attribute.args2) {
                            _template_body_html = $(`#${_attribute.args2}`).html();
                        }
                    }
                });
            }

            if (_template_head_html == null) {
                _template_head_html = $("#template_header").html();
            }
            var _modelhtml = template(_template_head_html, { list: o.properties, config: _config });
            //一级模型 转化成 二级模型
            if (_template_body_html == null) {
                var _template_body = $("#template_body").html();
                var _bodyhtml = template(_template_body, { list: o.properties, config: _config });
                _template_body_html = _bodyhtml.replace(/{{/g, '<%').replace(/}}/g, '%>');
            }
            $(".table").find("thead").html(_modelhtml);
            //处理查询项
            if (o.queryProperties) {
                _globdataQueryProperties = o.queryProperties;
                HandlerQueryItem(o.queryProperties);
            } else {
                _readpagedata(1);
            }
            //读取数据
        }
    });

```

看如上代码，就是先向后端获得这个页面的搜索区域的数据模型属性和下方表格的数据模型
然后



```
_readpagedata(1);

```

才是获取表格的数据，也就是说第二页之后的数据是只要请求一次的，只要首次打开才要获取数据模型的属性，如果你使用本地缓也是可以省略第一次的模型属性的数据的！
UI中再基于JS获取到的模型进行渲染



```
        
        

        
        

        
        

```

注意表格中的模板进行了二次转化，也就是获取模型后先转化成数据模型获得模板，然后再结合实际的页数据再一次进行UI渲染


## 表单页面的实现


表单页面打开后会判断是否是编辑，其实整个思路是一样的，只是请求的接口不一样，一个是XXXAddDto一个是UpdateDto



```
/**
 * 读取模型和默认值，值等
 */
function FuncFilexModel() {
    // console.log(_id);
    if (_id && _id != '0' && _id != 0) {
        _apiget(`/api/app/${_classPath}/${_id}/readUpdateModel`, true, (c, o) => {
            if (c == 200) {
                loadHeader(o);
                if (o.properties) {
                    LoadModelProperity(o.properties);
                }
                if (o.title) {
                    this.document.title = "更新" + o.title;
                }
            }
        });
    } else {
        _apiget(`/api/app/${_classPath}/readAddModel`, true, (c, o) => {
            if (c == 200) {
                loadHeader(o);
                if (o.properties) {
                    LoadModelProperity(o.properties);
                }
                if (o.title) {
                    this.document.title = "新增" + o.title;
                }
            }
        });
    }
}

```

其实2个请求到最后都是到LoadModelProperity的函数中



```
/**
 * 读取数据的模型 处理数据的模型
 * @param {*} properties 
 */
function LoadModelProperity(properties) {
    _modelProperties = properties;
    handlerExchangeDataTypeToUIType(properties);

    var _template = $("#templatemodel").html();
    var _ahtml = template(_template, { list: properties, config: _config });
    $(".paste-form-body").html(_ahtml);
    setTimeout(function () {
        FormLoaded(properties);
        funcAppendInputLength();

        //计算高度，返回
        var _height = $(".newform").height() + 140;
        if (_has_outer) {
            //如果计算的高度小于600，表单中有outer则至少保证高度为600
            if (_height < 600) {
                _height = 600;
            }
        }
        var _index = parent.layer.getFrameIndex(window.name); //获取窗口索引
        if (window.parent.set_dialog_height) {
            window.parent.set_dialog_height(_height, _index);
        } else {
            console.log('没有在父级找到函数set_dialog_height');
        }

        $(".ulselects").on('click', 'li', function () {
            if ($(this).hasClass('selected')) {
                $(this).removeClass('selected');
            } else {
                $(this).addClass('selected');
            }
        });


    }, 100);
}

```

然后是前端HTML中，基于JS获取的数据进行UI渲染



```
        

```

所以说，针对不同项目的不同需求，或者说你们的个人习惯，可以对上面的代码进行修改以便适应自己的项目，比如我的项目一般不会用到date,一般用到的是datetime，所以我就没考虑date的情况了！


下一次将介绍在实际中遇到的问题，和PasteForm是如何处理问题的


