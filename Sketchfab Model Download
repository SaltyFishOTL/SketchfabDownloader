// ==UserScript==
// @name         Sketchfab Model Download
// @version      2.12
// @description  download sketchfab models
// @author       great
// @match        *://*.sketchfab.com/*
// @require      https://lf9-cdn-tos.bytecdntp.com/cdn/expire-1-M/jszip/3.1.5/jszip.min.js
// @require      https://lf9-cdn-tos.bytecdntp.com/cdn/expire-1-M/jszip-utils/0.1.0/jszip-utils.min.js
// @require      https://lf9-cdn-tos.bytecdntp.com/cdn/expire-1-M/FileSaver.js/1.3.8/FileSaver.min.js
// @require      https://lf9-cdn-tos.bytecdntp.com/cdn/expire-1-M/jquery/3.6.0/jquery.min.js
// @require      https://lf9-cdn-tos.bytecdntp.com/cdn/expire-1-M/bootstrap-tagsinput/0.8.0/bootstrap-tagsinput.min.js
// @run-at       document-start
// @grant        unsafeWindow
// @grant        GM_download
// ==/UserScript==

var _DivideType = 0;
var _OpenDownload = 1;
var _OpenFav = 1;
var _ExtJpeg = 0;

var zip = new JSZip();
let folder = zip.folder('collection');

var button_dw = false;
var func_drawGeometry = /(this\._stateCache\.drawGeometry\(this\._graphicContext,t\))/g;
var fund_drawArrays = /t\.drawArrays\(t\.TRIANGLES,0,6\)/g;
var func_renderInto1 = /x\.renderInto\(n,S,y/g;
var func_renderInto2 = /g\.renderInto=function\(e,i,r/g;
var func_getResourceImage = /getResourceImage:function\(e,t\){/g;


var _FreeMemory = 0;
var addbtnfunc;
var createPreview, createLinkFile, getUri;
var gr_id, gr_url, gr_imgUrl, gr_imgName, gr_texNums, gr_info, dl_btn, dl_title, dl_status, timeID, objNums, objNamePext={}, mtl="", previewName="预览图.jpeg", linkFileName="快捷方式.url", imgNamePextList={};

var _disPlayMod = 1;
var mid, url, freedl, list, favList={}, favType={}, webUrl=location.href, webSite, colNums, colTop, searchPage=1;

if (webUrl.indexOf('/models/') > -1) {
    webSite = 'page_models';
} else if (webUrl.indexOf('/3d-models/popular') > -1) {
    webSite = 'page_popular';
}

if (webSite == 'page_models' && window.top.document.body.clientWidth > 1500) window.top.document.body.style.width = '1500px';


(function() {
    'use strict';
    if(webSite != 'page_models' && !_OpenDownload) return;

    var window = unsafeWindow;

    window.allmodel = [];
    var saveimagecache1 = {};
    var saveimagecache2 = {};
    var objects = {};

    var gr_iframe;

    let gr_canvas = document.createElement('canvas');
    gr_canvas.width = 100;
    gr_canvas.height = 100;
    let gr_context = gr_canvas.getContext('2d');
    let gr_temp,gr_data,gr_imageData;


    window.addEventListener('message',function(e){
        if (e.data.status == 6) {
            var res = e.data.modelInfo;
            gr_imgName = previewName;
            gr_imgUrl = res.imgUrl;
            gr_url = res.viewerUrl;
            gr_texNums = res.textureCount;
            createPreview(gr_imgUrl, gr_imgName, "jpeg", 1);
            createLinkFile(gr_url, linkFileName);
            if(dl_status) dl_status.innerHTML = "Please wait... /tex: " + gr_texNums;
        }
    });

    getUri = function() {
        gr_id = location.href.replace(/\?.+/g,'');
        if (gr_id.indexOf("sketchfab.com/models") > -1) {
            gr_id = gr_id.replace(/.+\/models\/([^\/]+?)\/.+/g,'$1');
        } else {
            gr_id="";
            return;
        }
        var data = {"postType": "getModelInfo", "postData": {"member": _ACCOUNT, "mid":gr_id}};
        gr_iframe.contentWindow.postMessage(data,'*');
    }

    createPreview = function(url, fileName, mimeType, type) {
        if (mimeType == "jpg") mimeType = "jpeg";
        var image = new Image();
        image.src = url;
        image.setAttribute("crossOrigin", "Anonymous");
        image.onload = function() {
            var canvas = document.createElement("canvas");
            var setW = type == 1 ? image.width : 300;
            var setH = type == 1 ? image.height : 300;
            canvas.width = setW;
            canvas.height = setH;
            var context = canvas.getContext("2d");
            context.drawImage(image, 0, 0, setW, setH);
            canvas.toBlob(function(blob){objects[fileName] = blob;},"image/" + mimeType);
        };
    }

    createLinkFile = function(linkURL, fileName) {
        var content = `[{000214A0-0000-0000-C000-000000000046}]
Prop3=19,11
[InternetShortcut]
IDList=
URL=${linkURL}
IconFile=
IconIndex=1`;
        var blob = new Blob([content]);
        objects[fileName] = blob;
    }

    var showStatus = function() {
        dl_title.innerHTML = "Packing... ";
        setTimeout(()=>{dl_title.innerHTML = "";}, 250);
    }

    var reImgName = function(oldName, uid) {
        var namePext = oldName.replace(/\.[^\.]+?$/g,"");
        var namePextL = namePext.toLowerCase();
        if (!imgNamePextList[namePextL]) {
            imgNamePextList[namePextL] = namePextL;
            return oldName;
        } else {
            namePext = namePext + "_" + uid;
            var ext = oldName.split(".").pop();
            var newName = namePext + "." + ext;
            return newName;
        }
    }

    var saveimage_to_list = function(url,file_name)
    {
        if (!saveimagecache2[url])
        {
            var mdl = {
                name: file_name
            }

            saveimagecache2[url] = mdl;
        }
    }

    addbtnfunc = function() {
        var p = document.evaluate("//div[@class='titlebar']", document, null, 9, null).singleNodeValue;
        if(p && !button_dw) {
            var btn = document.createElement("a");
            btn.setAttribute("id", "dl_btn");
            btn.innerHTML = "<pre style='color:#ff3e3e; text-shadow: 0 0 3px #454545; font-family: Arial; font-weight: bold;'>DOWNLOAD</pre>";
            btn.addEventListener("click", dodownload , false);
            p.appendChild(btn);
            // add status info
            var info = document.createElement("div");
            info.setAttribute("class", "dl_info");
            info.style.cssText = "color: #00d500; text-shadow: 0 0 3px #000000; z-index: 99; line-height: 1.5em; margin-top: 30px; font-size: 12px; font-weight: bold; letter-spacing: 1px; position: relative; text-align: right; padding-right: 10px; padding-left:10px;";
            info.innerHTML = "<span id='dl_title'></span><span id='dl_status'></span>";
            p.parentNode.insertBefore(info,p.nextSibling);
            gr_info = info;
            dl_btn = document.getElementById("dl_btn");
            dl_title = document.getElementById("dl_title");
            dl_status = document.getElementById("dl_status");
            dl_status.style.cssText = "background:#1fb10d; border-radius: 50px; padding: 0 10px; color:#fff;";
            button_dw = true;
            // add iframe
            gr_iframe = document.createElement("iframe");
            gr_iframe.setAttribute("id", "gr_iframe");
            gr_iframe.setAttribute("height", "10");
            gr_iframe.setAttribute("border", "0");
            gr_iframe.setAttribute("frameborder", "0");
            gr_iframe.setAttribute("src", "https://www.gieex.com/skf/iframe.html");
            document.body.appendChild(gr_iframe);
        } else {
            setTimeout(addbtnfunc, 3000);
        }
    }

    var dodownload = function() {
        dl_btn.removeEventListener('click', dodownload, false);
        dl_btn.innerHTML = "";
        if (Object.keys(saveimagecache2).length > 0 && Object.keys(saveimagecache1).length != Object.keys(saveimagecache2).length && _DivideType != 1) {
            Object.keys(saveimagecache2).forEach(function(url) {
                if (!saveimagecache1[url]) {
                    var ext;
                    if (_ExtJpeg == 1) {
                        ext = "jpeg";
                    } else {
                        ext = url.split(".").pop();
                    }
                    var ret = saveimagecache2[url].name.split(".").pop();
                    saveimagecache2[url].name = saveimagecache2[url].name.replace('.'+ret,'');
                    var name = saveimagecache2[url].name + "." + ext;
                    saveimagecache2[url].name = name;
                    createPreview(url, name, ext, 1);
                }
            });
        }
        objNums = window.allmodel.length;
        dl_title.innerHTML = "Packing... ";
        timeID = setInterval(showStatus, 500);

        if (_DivideType == 2) {
            if(dl_status) dl_status.innerHTML = "tex: " + Object.keys(saveimagecache2).length + " / obj: "+ objNums;
            setTimeout(PackAll, 3000);
            return;
        }

        var idx = 1;
        window.allmodel.forEach(function(obj)
                                {
            var objName;
            if (obj._name) {
                objName = obj._name.split('_');
                objName = objName.length > 1 ? objName[objName.length-2] : objName[0];
                if (objName == "") objName = "0";
                var objNameL = objName.toLowerCase();
                if (!objNamePext[objNameL]) {
                    objNamePext[objNameL] = 1;
                } else {
                    objNamePext[objNameL] += 1;
                    objName = objName + "_" + objNamePext[objNameL];
                }
            } else {
                objName = "model_"+idx;
            }
            objName = String(objName);
            var mdl = {
                name: objName,
                obj: parseobj(obj),
                tex: parsetex(obj,idx)
            }
            dosavefile(mdl,idx);
            idx++;
        });

        setTimeout(PackAll, 3000);
    }

    var PackAll = function ()
    {
        for (var obj in objects) {
            var ext = obj.split(".").pop();
            if((ext != "png" && ext != "jpg" && ext != "jpeg" && ext != "bmp" && ext != "gif" && ext != "tga" && ext != "tif") || obj == previewName) {
                folder.file(obj, objects[obj], {binary:true});
            } else {
                folder.file("tex\\" + obj, objects[obj], {binary:true});
            }
        }

        var file_name = document.getElementsByClassName('model-name__label')[0].textContent + " -" + gr_id;
        if (objects[gr_imgName]) {
            saveAs(objects[gr_imgName], file_name+".jpeg");
        }
        folder.generateAsync({ type: "blob" }).then(content => {saveAs(content, file_name + ".zip"); gr_info.innerHTML = '<span style="color: #08f508; text-shadow: 0 0 3px #000000;">文件大小：'+(content.size/1024/1024).toFixed(1)+' MB</span>';});
        clearInterval(timeID);
        gr_info.innerHTML = '<span style="color: #08f508; text-shadow: 0 0 3px #000000;">Successful!</span>';
        var mid = window.top.location.href.split("-").pop();
        window.top.document.getElementById('gr_iframe').contentWindow.postMessage({"postType": "setDownLoad", "postData":{"member": _ACCOUNT, "mid": mid}},'*');
    }

    var parseobj = function(obj)
    {
        var list = [];
        obj._primitives.forEach(function(p) {
            if(p && p.indices) {
                list.push({
                    'mode' : p.mode,
                    'indices' : p.indices._elements
                });
            }
        })

        var attr = obj._attributes;
        return {
            vertex: attr.Vertex._elements,
            normal: attr.Normal ? attr.Normal._elements : [],
            uv: attr.TexCoord0 ? attr.TexCoord0._elements :
            attr.TexCoord1 ? attr.TexCoord1._elements :
            attr.TexCoord2 ? attr.TexCoord2._elements :
            attr.TexCoord3 ? attr.TexCoord3._elements :
            attr.TexCoord4 ? attr.TexCoord4._elements :
            attr.TexCoord5 ? attr.TexCoord5._elements :
            attr.TexCoord6 ? attr.TexCoord6._elements :
            attr.TexCoord7 ? attr.TexCoord7._elements :
            attr.TexCoord8 ? attr.TexCoord8._elements : [],
            primitives: list,
        };
    }

    var textype = {
        DiffusePBR: "map_Kd",
        DiffuseColor: "map_Kd",
        SpecularPBR: "map_Ks",
        SpecularColor: "map_Ks",
        GlossinessPBR: "map_Pm",
        NormalMap: "map_Bump",
        EmitColor: "map_Ke",
        AlphaMask: "map_d",
        Opacity: "map_o",
        AlbedoPBR: "map_Kd",
        RoughnessPBR: "map_Ns",
        MetalnessPBR: "map_Ks",
        AOPBR: "map_Ka",
        BumpMap: "map_Bump",
        DiffuseIntensity: "map_Ka"
    };
    var fixTextype = function(texStr) {
        if (texStr.indexOf('Specular') > -1) {
            return 'SpecularColor';
        }
        return texStr;
    }

    var parsetex = function(obj,idx) {
        var texlist = [];
        var stateset = obj._parents[0].stateset ? obj._parents[0].stateset : obj.stateset;
        if (stateset && stateset._textureAttributeArrayList) {
            stateset._textureAttributeArrayList.forEach(function(arr,k) {
                if (arr && arr[0]&& arr[0]._object) {
                    var object = arr[0]._object;
                    var channelNums = object._channels.length || 0;
                    if (channelNums == 1) {
                        var image = object._texture && object._texture._imageModel;
                        if (image) {
                            var fName;
                            if (object._texture._image._url) {
                                fName = saveimagecache2[object._texture._image._url].name;
                            } else {
                                image.attributes.images.forEach(function(img) {
                                    if (saveimagecache2[img.url]) {
                                        fName = saveimagecache2[img.url].name;
                                    }
                                });
                            }
                            var texStr = fixTextype(object._channels[0]);
                            var type = textype[texStr] || object._channelName || "map_Kd";
                            texlist.push({
                                url: "none",
                                type: type,
                                filename: fName
                            });
                        }
                    } else if (channelNums > 1) {
                        var packed = object._packedTextures;
                        if (packed) {
                            object._channels.forEach(function(channel) {
                                var image = packed[channel] && packed[channel].texture && packed[channel].texture._imageModel;
                                if (image) {
                                    var fName;
                                    if (packed[channel].texture._image._url) {
                                        fName = saveimagecache2[packed[channel].texture._image._url].name;
                                    } else {
                                        image.attributes.images.forEach(function(img) {
                                            if (saveimagecache2[img.url]) {
                                                fName = saveimagecache2[img.url].name;
                                            }
                                        });
                                    }
                                    var texStr = fixTextype(channel);
                                    var type = textype[texStr] || "map_Kd";
                                    texlist.push({
                                        url: "none",
                                        type: type,
                                        filename: fName
                                    });
                                }
                            });
                        }
                    }
                }
            });
        }
        return texlist;
    }

    var dosavefile = function(mdl,idx)
    {
        var obj = mdl.obj;
        var vtNums = obj.uv.length ? obj.uv.length : 0;

        var str = '';
        str += 'mtllib Texture.mtl\n';
        str += 'o ' + mdl.name + '\n';
        for (var i = 0; i < obj.vertex.length; i += 3) {
            str += 'v ';
            for (var j = 0; j < 3; ++j) {
                str += obj.vertex[i + j].toFixed(6) + ' ';
            }
            str += '\n';
        }
        for (i = 0; i < obj.normal.length; i += 3) {
            str += 'vn ';
            for (j = 0; j < 3; ++j) {
                str += obj.normal[i + j].toFixed(6) + ' ';
            }
            str += '\n';
        }

        for (i = 0; i < obj.uv.length; i += 2) {
            str += 'vt ';
            for (j = 0; j < 2; ++j) {
                str += obj.uv[i + j].toFixed(6) + ' ';
            }
            str += '\n';
        }
        if (vtNums > 0) str += 'usemtl ' + mdl.name + '\n';
        str += 's on \n';

        var vn = obj.normal.length != 0;
        var vt = obj.uv.length != 0;

        for (i = 0; i < obj.primitives.length; ++i) {
            var primitive = obj.primitives[i];
            if (primitive.mode == 4 || primitive.mode == 5) {
                var strip = (primitive.mode == 5);
                for (j = 0; j + 2 < primitive.indices.length; !strip ? j += 3 : j++) {
                    str += 'f ';
                    var order = [ 0, 1, 2];
                    if (strip && (j % 2 == 1)) {
                        order = [ 0, 2, 1];
                    }
                    for (var k = 0; k < 3; ++k)
                    {
                        var faceNum = primitive.indices[j + order[k]] + 1;
                        str += faceNum;
                        if (vn || vt) {
                            str += '/';
                            if (vt) {
                                str += faceNum;
                            }
                            if (vn) {
                                str += '/' + faceNum;
                            }
                        }
                        str += ' ';
                    }
                    str += '\n';
                }
            }
            else {
            }
        }
        str += '\n';
        var objblob = new Blob([str], {type:'text/plain'});
        objects[mdl.name+".obj"] = objblob;

        if (vtNums > 0) {
            var tex = mdl.tex;
            mtl += 'newmtl ' + mdl.name + '\n';
            tex.forEach(function(texture) {
                mtl += texture.type + ' tex\\\\' + texture.filename + '\n';
            });
            mtl += '\n';
        }
        if (idx == objNums) {
            var mtlblob = new Blob([mtl], {type:'text/plain'});
            objects["Texture.mtl"] = mtlblob;
        }
        if(dl_status) dl_status.innerHTML = "tex: " + Object.keys(saveimagecache2).length + " / obj: "+ objNums;

    }


    window.attachbody = function(obj)
    {
        if(obj._faked != true && ((obj.stateset && obj.stateset._name) || obj._name || (obj._parents && obj._parents[0]._name)) ) {
            obj._faked = true;
            if(obj._name == "composer layer" || obj._name == "Ground - Geometry") return;
            window.allmodel.push(obj);
        }
    }


    window.drawhookcanvas = function(e, imagemodel)
    {
        var flag = (new Error()).stack.split("\n")[3].trim().split(" ")[1];
        if(flag == "getImageSmallest")
        {
            return e;
        }
        if(imagemodel)
        {
            let alpha = e.options && e.options.format ? e.options.format : 'R';
            let filename_image = imagemodel.attributes.name;
            if (filename_image == "internal_ground_ao_texture.jpeg") {
                return e;
            }
            let uid = imagemodel.attributes.uid;
            let url_image = e.url;
            let max_size = 0;
            let obr = e;
            imagemodel.attributes.images.forEach(function(img)
                                                 {
                let alpha_is_check;
                if (img.options && img.options.format) {
                    alpha_is_check = alpha == "A" ? img.options.format == alpha : true;
                } else {
                    alpha_is_check = true;
                }

                let d = img.width;
                if (d % 2 != 0 && d > 1) {
                    d = d - 1;
                }
                while ( d % 2 == 0 )
                {
                    d = d / 2;
                }
                let target = 0;
                if (img.options && img.options.format) {
                    if(img.size > max_size && alpha_is_check && d == 1 && img.options.format) {
                        target = 1;
                    }
                } else if (img.size == e.size && img.url == e.url) {
                    target = 1;
                }
                if (target == 1) {
                    max_size = img.size;
                    url_image = img.url;
                    uid = img.uid;
                    obr = img;
                }
            });

            var ext;
            if (_ExtJpeg == 1) {
                ext = "jpeg";
            } else {
                ext = url_image.split(".").pop();
                if(ext != "png" && ext != "jpg" && ext != "jpeg" && ext != "bmp" && ext != "gif" && ext != "tga" && ext != "tif") {
                    ext = "png";
                }
            }
            filename_image = filename_image.replace(/\.[^\.]+?$/g,"."+ext);

            if(!saveimagecache2[url_image])
            {
                let newName = reImgName(filename_image, uid);
                saveimage_to_list(url_image, newName);
            }

            return obr;
        }
        return e;
    }

    window.drawhookimg = function(gl,t)
    {
        var url = t[5].currentSrc;
        var width = t[5].width;
        var height = t[5].height;

        if(!saveimagecache2[url])
        {
            return;
        }

        saveimagecache1[url] = saveimagecache2[url];

        gr_data = new Uint8Array(width * height * 4);
        gl.readPixels(0, 0, width, height, gl.RGBA, gl.UNSIGNED_BYTE, gr_data);

        var halfHeight = height / 2 | 0;
        var bytesPerRow = width * 4;

        gr_temp = new Uint8Array(width * 4);
        for (let y = 0; y < halfHeight; ++y)
        {
            let topOffset = y * bytesPerRow;
            let bottomOffset = (height - y - 1) * bytesPerRow;

            gr_temp.set(gr_data.subarray(topOffset, topOffset + bytesPerRow));

            gr_data.copyWithin(topOffset, bottomOffset, bottomOffset + bytesPerRow);

            gr_data.set(gr_temp, bottomOffset);
        }

        gr_canvas.width = width;
        gr_canvas.height = height;
        gr_context.clearRect(0, 0, width, height);

        gr_imageData = gr_context.createImageData(width, height);
        gr_imageData.data.set(gr_data);
        gr_context.putImageData(gr_imageData, 0, 0);

        var name = saveimagecache2[url].name;
        var ext = name.split(".").pop();

        gr_canvas.toBlob(function(blob){
            objects[name] = blob;
            if (Object.keys(saveimagecache1).length == Object.keys(saveimagecache2).length && _FreeMemory == 1) {
                var canvas2 = document.getElementsByClassName('canvas')[0];
                var gl2 = canvas2.getContext('webgl');
                gl2.getExtension('WEBGL_lose_context').loseContext();
            }
        },"image/"+ext);

        gr_imageData = null;
        gr_data = null;
        gr_temp = null;
        if(dl_status) dl_status.innerHTML = String("tex: " + Object.keys(saveimagecache1).length + "/" + Object.keys(saveimagecache2).length);

    }

})();

(() => {
    "use strict";
    if(webSite != 'page_models' && !_OpenDownload) return;

    const Event = class {
        constructor(script, target) {
            this.script = script;
            this.target = target;

            this._cancel = false;
            this._replace = null;
            this._stop = false;
        }

        preventDefault() {
            this._cancel = true;
        }
        stopPropagation() {
            this._stop = true;
        }
        replacePayload(payload) {
            this._replace = payload;
        }
    };

    let callbacks = [];
    window.addBeforeScriptExecuteListener = (f) => {
        if (typeof f !== "function") {
            throw new Error("Event handler must be a function.");
        }
        callbacks.push(f);
    };
    window.removeBeforeScriptExecuteListener = (f) => {
        let i = callbacks.length;
        while (i--) {
            if (callbacks[i] === f) {
                callbacks.splice(i, 1);
            }
        }
    };

    const dispatch = (script, target) => {
        if (script.tagName !== "SCRIPT") {
            return;
        }

        const e = new Event(script, target);

        if (typeof window.onbeforescriptexecute === "function") {
            try {
                window.onbeforescriptexecute(e);
            } catch (err) {
                console.error(err);
            }
        }

        for (const func of callbacks) {
            if (e._stop) {
                break;
            }
            try {
                func(e);
            } catch (err) {
                console.error(err);
            }
        }

        if (e._cancel) {
            script.textContent = "";
            script.remove();
        } else if (typeof e._replace === "string") {
            script.textContent = e._replace;
        }
    };
    const observer = new MutationObserver((mutations) => {
        for (const m of mutations) {
            for (const n of m.addedNodes) {
                dispatch(n, m.target);
            }
        }
    });
    observer.observe(document, {
        childList: true,
        subtree: true,
    });
})();

(() => {
    "use strict";
    if(webSite != 'page_models' && !_OpenDownload) return;

    window.onbeforescriptexecute = (e) => {
        var links_as_arr = Array.from(e.target.childNodes);

        links_as_arr.forEach(function(srimgc)
                             {
            if(srimgc instanceof HTMLScriptElement)
            {
                if (srimgc.src.indexOf("web/dist/") >= 0 || srimgc.src.indexOf("standaloneViewer") >= 0)
                {
                    e.preventDefault();
                    e.stopPropagation();
                    var req = new XMLHttpRequest();
                    req.open('GET', srimgc.src, false);
                    req.send('');
                    var jstext = req.responseText;
                    var ret = func_renderInto1.exec(jstext);

                    if (ret && 0)
                    {
                        var index = ret.index + ret[0].length;
                        var head = jstext.slice(0, index);
                        var tail = jstext.slice(index);
                        jstext = head + ",i" + tail;
                    }

                    ret = func_renderInto2.exec(jstext);

                    if (ret && 0)
                    {
                        var index = ret.index + ret[0].length;
                        var head = jstext.slice(0, index);
                        var tail = jstext.slice(index);
                        jstext = head + ",image_data" + tail;
                    }

                    ret = fund_drawArrays.exec(jstext);

                    if (ret && 0)
                    {
                        var index = ret.index + ret[0].length;
                        var head = jstext.slice(0, index);
                        var tail = jstext.slice(index);
                        jstext = head + ",window.drawhookimg(t,image_data)" + tail;
                    }

                    ret = func_getResourceImage.exec(jstext);

                    if (ret)
                    {
                        var index = ret.index + ret[0].length;
                        var head = jstext.slice(0, index);
                        var tail = jstext.slice(index);
                        jstext = head + "e = window.drawhookcanvas(e,this._imageModel);" + tail;
                    }

                    ret = func_drawGeometry.exec(jstext);
                    if (ret) {
                        setTimeout(addbtnfunc, 3000);
                        setTimeout(getUri, 4000);
                    }

                    if (ret && _DivideType != 2)
                    {
                        var index1 = ret.index + ret[1].length;
                        var head1 = jstext.slice(0, index1);
                        var tail1 = jstext.slice(index1);
                        jstext = head1 + ";window.attachbody(t);" + tail1;
                    }

                    var idx = 0;
                    var obj = document.createElement('script');
                    obj.type = "text/javascript";
                    obj.text = jstext;
                    document.getElementsByTagName('head')[0].appendChild(obj);
                }
            }
        });
    };
})();


(function($) {
    'use strict';
    if(webSite == 'page_models' && !_OpenFav) return;

    $(function(){
        var gr_css = '<style type="text/css" index="gr_css">\
            .gr_cicle {display: block; width:30px; height:30px; border: 5px solid #ff5722; position: absolute; margin-top: -40px; margin-left: 10px; border-radius: 50%;}\
            .gr_fav {color:#9e9e9e; padding: 10px; margin-left: -10px; margin-top: -10px; cursor: pointer;}\
            .gr_fav:hover {color:#ff5722; }\
            .gr_fav::before {content: "\\f05d" " "; font-family: Font Awesome\\ 6 Pro; font-size: 20px;}\
            .gr_nums {position: absolute; right:10px; margin-top: -22px; color:#fff; font-size: 12px; font-style: italic; text-shadow: 0 0 5px #000; display: none;}\
            .gr_delPre {cursor: pointer; margin-right: 10px; color:#999; font-size: 13px;}\
            .gr_delPre:hover {color:#ff5722; font-weight: bold;}\
            #gr_tag {z-index:999; background: #ffffff; padding: 20px; border-radius: 20px; border: 2px solid #1abc9c;position: absolute; margin-left:-1000px; opacity: 0; width:500px; box-shadow: 0px 1px 4px #959595; transition: .25s linear;}\
            #gr_tagbg {z-index:888; background: rgba(0,0,0,0.5);; position: fixed; top:0; left:0; width:100%; height:100%; transition: .25s linear;}\
            #gr_edit {float: right; cursor: pointer;}\
            #gr_setdl {cursor: pointer; font-size:13px; color:#999; margin-left:10px;}\
            #gr_edit:hover, #gr_setdl:hover {color: #FF6651;}\
            #gr_menu {background: #e50707;display: inline-block;padding: 0 7px;font-weight: bold; color: #fff;border-radius: 4px;cursor: pointer; margin-left: 10px;}\
            #gr_menu:hover {background: #ff3f3f;}\
            #gr_main_1 {width: 600px; height: 100%; background: #2b3340; padding: 20px; color: #999; position: fixed; top: 0; right:0; display: none;z-index:998; box-shadow: 3px 0 9px #005fab;}\
            #gr_main_2 {width: 570px; height: 100%; background: #303948; padding: 20px 10px; color: #999; position: fixed; top: 0; right:0; display: none;z-index:990; box-shadow: 3px 0 9px #005fab; overflow-y: auto;}\
            #gr_main_2 .model-smallcard__thumbnail {width: 230px; display: inline-block; margin: 10px;}\
            .grdl_tab_2::-webkit-scrollbar, .grdl_finished::-webkit-scrollbar, #grdl_tab_1 textarea::-webkit-scrollbar, #grdl_fav::-webkit-scrollbar, #gr_main_2::-webkit-scrollbar {width: 10px; height: 1px;}\
            .grdl_tab_2::-webkit-scrollbar-thumb, .grdl_finished::-webkit-scrollbar-thumb, #grdl_tab_1 textarea::-webkit-scrollbar-thumb, #grdl_fav::-webkit-scrollbar-thumb, #gr_main_2::-webkit-scrollbar-thumb {background: #1c3254; border:1px solid rgba(35,74,137,0.66); border-right:none;}\
            .grdl_tab_2::-webkit-scrollbar-track, .grdl_finished::-webkit-scrollbar-track, #grdl_tab_1 textarea::-webkit-scrollbar-track, #grdl_fav::-webkit-scrollbar-track, #gr_main_2::-webkit-scrollbar-track {background: #242d3c;}\
            #grdl_tab_1 textarea::-webkit-resizer {background: #242d3c;}\
            #gr_title {color: #03a9f4; font-size: 19px; font-weight: bold; margin-bottom: 10px;}\
            #gr_main ul {list-style-type: none; padding-left:45px;}\
            #gr_main li {position: relative;}\
            #gr_main a {text-decoration: none; display: block; padding: 5px; color: #999;}\
            #gr_main a:hover {text-decoration: underline;}\
            #gr_nav {margin-bottom: 10px;}\
            .nav_tab {display: inline-block; cursor: pointer; padding: 0 20px; color: #03a9f4; border: 1px solid #185a8f; border-right: 0;}\
            .nav_tab.selected, .nav_tab:hover {background: #0b4c81;}\
            .nav_tab:last-of-type {border-right: 1px solid #185a8f;}\
            .grdl_tab {display:none;}\
            .grdl_btn_fav {display: block; padding: 10px; margin: 0 auto; margin-top:50%; cursor: pointer; width: 180px; height: 60px; background: #005fab; color: #fff; border: 1px solid #2e86cd;}\
            .grdl_btn_fav:hover {background: #1e84d5;}\
            #grdl_fav {width: 100%; overflow-y: auto;}\
            .grdl_fav_tab {display: inline-block; padding:5px 10px; margin: 0 10px 10px 0; font-size:13px; cursor: pointer;}\
            .grdl_fav_tab:hover {text-decoration: underline; color: #03a9f4;}\
            .grdl_fav_i {color: #607d8b;}\
            .grdl_fav_color {color: #03a9f4;}\
            .grdl_fav_n {font-size: 15px;}\
            .grdl_fav_a {}\
            #grdl_tab_1 {width: 450px; height:60px; vertical-align: top; display: inline-block; position: relative;}\
            #grdl_tab_1 textarea {width: 440px; height: 60px; color: #999; padding: 5px; background: none; border: 1px solid #455062; z-index: 920; position: absolute; top: 0; font-size:12px;}\
            #grdl_tips_1 {position: absolute; top: 0; padding: 5px; z-index: 910; font-size: 14px;}\
            .grdl_tab_2 {width: 550px; height: 420px; overflow-y: auto; border: 1px solid #455062; color: #999; margin-top: 10px;}\
            #grdl_tab_btn {vertical-align: top; margin-left: 6px; display: inline-block;}\
            .grdl_btn_list {padding: 10px; cursor: pointer;width: 90px; height: 60px; background: #005fab; color: #fff; border: 1px solid #2e86cd;}\
            .grdl_btn_list:hover {background: #2196f3;}\
            .grdl_tab_2 li:first-child {border-bottom: 1px solid #03a9f4; margin-bottom:40px; margin-top: 20px;}\
            .grdl_tab_2 li:first-child a {padding: 15px 0; color: #03a9f4;}\
            .grdl_finished {width: 550px; height: 120px; color: #999; font-size: 13px; border: 1px solid #455062; overflow-y: auto; margin-top: 20px;}\
            .grdl_finished p {padding-left: 10px;}\
            .grli_num {display: inline-block; position: absolute; left: -35px; top: 8px; font-size: 13px;}\
            .gr_color {color: #ff5722 !important;}\
            .gr_colorG {color: #00d500 !important; font-weight: bold;}\
            .gr_colorbg {background: #ff5722 !important;}\
            .freedl_color {color: #8bc34a !important;}\
            .gr_tagborder {border:2px solid #fd9270;}\
            .gr_display {display: inline-block !important;}\
            .selected {display: inline-block !important;}\
            .selected_bd {border: 1px solid #03a9f4; border-radius: 30px;}\
\
.bootstrap-tagsinput { background-color: #fff; border: 1px solid #ccc; box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075); display: inline-block; padding: 4px 6px; color: #555; vertical-align: middle; border-radius: 4px; max-width: 100%; line-height: 22px; cursor: text; }\
.bootstrap-tagsinput input { border: none; box-shadow: none; outline: none; background-color: transparent; padding: 0 6px; margin: 0; width: auto; max-width: inherit; }\
.bootstrap-tagsinput.form-control input::-moz-placeholder { color: #777; opacity: 1; }\
.bootstrap-tagsinput.form-control input:-ms-input-placeholder { color: #777; }\
.bootstrap-tagsinput.form-control input::-webkit-input-placeholder { color: #777; }\
.bootstrap-tagsinput input:focus { border: none; box-shadow: none; }\
.bootstrap-tagsinput .tag { margin-right: 2px; color: white; }\
.bootstrap-tagsinput .tag [data-role="remove"] { margin-left: 8px; cursor: pointer; }\
.bootstrap-tagsinput .tag [data-role="remove"]:after { content: "x"; padding: 0px 2px; }\
.bootstrap-tagsinput .tag [data-role="remove"]:hover { box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.2), 0 1px 2px rgba(0, 0, 0, 0.05); }\
.bootstrap-tagsinput .tag [data-role="remove"]:hover:active { box-shadow: inset 0 3px 5px rgba(0, 0, 0, 0.125); }\
\
            .bootstrap-tagsinput {width: 100%; border: none; padding: 10px; box-shadow:none;}\
            .bootstrap-tagsinput .tag [data-role="remove"] {display: none;}\
            .bootstrap-tagsinput .tag [data-role="remove"]:hover {line-height: 15px; font-size: 20px; color:#000000; margin-left: 5px;}\
            .bootstrap-tagsinput .tag {background-color: #1abc9c; border-radius: 4px; font-size: 13px; overflow: hidden; display: inline-block; padding: 3px 8px; white-space: nowrap; cursor: pointer; transition: all 0.2s linear;}\
            .bootstrap-tagsinput .tag:hover {background-color: #FF6651;}\
            .bootstrap-tagsinput input {vertical-align: top; height: 28px; font-size: 14px; color: #34495e; min-width: 130px;}\
        </style>';

        $('head').append(gr_css);
        $('body').append('<iframe id="gr_iframe" name="gr_iframe" src="https://www.gieex.com/skf/iframe.html" width="200" height="10" border="0" frameborder="0" align="center"></iframe>');
        $('body').append('<div id="gr_tag"><h2 style="display: inline-block;">添加到收藏夹</h2><a id="gr_setdl">(设为已下载)</a><span id="gr_edit" data-edit="0">编辑</span><input data-role="tagsinput" id="tagsinputval" name="tagsinput" placeholder="输入新分类后回车" value=""/><div id="gr_tagList"><ul></ul></div></div>');

        if ($('body').hasClass('model-page')) {
            $('body').append('<a id="gr_close" style="position: fixed; right: 30px; top: 70px;background: #e50707;display: inline-block;padding: 12px 7px;font-weight: bold;font-size: 21px;color: #fff;border-radius: 4px;cursor: pointer;" onclick="window.close();">Close</a>');
        }
    });

    window.onload = function () {

        var gr_iframe = document.getElementById('gr_iframe');

        $('#tagsinputval').tagsinput({
            trimValue: true
        });
        $('#tagsinputval').on('itemAdded', function(event) {
            if ($('#gr_edit').data('edit') == '1') {
                $('.bootstrap-tagsinput .tag [data-role="remove"]').last().addClass('gr_display');
            }
        });
        $('#tagsinputval').on('beforeItemRemove', function(event) {
            if(!confirm('确定要删除该分类吗？删除后该分类下的数据将自动清空')) {
                event.cancel = true;
                return;
            }
            var data = {"postType": "delType", "postData": {"member": _ACCOUNT, "type": event.item}};
            gr_iframe.contentWindow.postMessage(data,'*');
        });
        $('#tagsinputval').on('itemRemoved', function(e) {
            e.stopPropagation();
        });
        $('body').on('click', '.bootstrap-tagsinput .tag [data-role="remove"]', function(e) {
            e.stopPropagation();
        });
        $('body').on('click', '#gr_tagbg', function(e) {
            $('#gr_tag').removeAttr('style');
            $('#gr_main_1').slideUp('fast');
            $('#gr_main_2').animate({right:'0'},'fast').hide('fast');
            $(this).remove();
            $('html').removeAttr('style');
            e.stopPropagation();
        });
        $('body').on('click', '#gr_main_1, #gr_main_2', function(e) {
            $('#gr_tag').removeAttr('style');
            e.stopPropagation();
        });
        $('body').on('click', '#gr_edit', function() {
            if($(this).data('edit') == '0'){
                $(this).data('edit','1');
                $(this).text('[ 编辑 ]');
            } else {
                $(this).data('edit','0');
                $(this).text('编辑');
            }
            $('.bootstrap-tagsinput .tag [data-role="remove"]').toggleClass('gr_display');
            $(this).toggleClass('gr_color');
        });
        $('body').on('click', '#gr_tag', function(e) {
            e.stopPropagation();
        });
        $('body').on('click', '#gr_setdl', function() {
            gr_iframe.contentWindow.postMessage({"postType": "setDownLoad", "postData":{"member": _ACCOUNT, "mid": mid}},'*');
        });

        $('body').on('click', function() {
            if (!list) return;
            $('body').find('.c-grid__item').each(function(){
                if ($('.gr_fav',this).length > 0) return;
                $('.card__main__corner.--top-left',this).append('<span class="gr_fav"> </span>');
                var cardNums_last = $(this).prev().find('.gr_nums').text() || 0;
                var cardNums = Number(cardNums_last)+1;
                if (location.href.indexOf('/search?') > -1 && searchPage) getcol(1);
                var show = cardNums % colNums == 0 ? ' selected' : '';
                $('.card__main__corner.--top-right',this).after('<span class="gr_nums'+show+'">'+cardNums+'</span>');
                $('.card__footer__right',this).prepend('<span class="gr_delPre">&lt;&lt;&lt;&lt;&lt;</span>');
                var mid = $('.card-model',this).length==1 ? $('.card-model',this).data('uid') : $('.model-smallcard',this).data('uid');
                if (list[mid]) {
                    if (_disPlayMod == 1) {
                        var color = list[mid].downloaded == 0 ? 'gr_color' : 'gr_colorG';
                        $('.gr_fav',this).addClass(color).data('favid',list[mid].fav_type_id);
                    } else {
                        $(this).remove();
                    }
                }
            });
        });
        var getcol = function (runOnce) {
            if (runOnce == 1) searchPage = 0;
            $('body').find('.c-grid__items .c-grid__item').each(function(k,v){
                if (k == 0) {
                    colTop = $(this).offset().top;
                    colNums = 1;
                } else {
                    if (colTop == $(this).offset().top) {
                        colNums += 1;
                    } else {
                        return false;
                    }
                }
            });
        }
        getcol();
        $('body').find('.c-grid__item').each(function(){
            $('.card__main__corner.--top-left',this).append('<span class="gr_fav"> </span>');
            if ($('.card-model',this).length==1) {
                var cardNums = $(this).index()+1;
                var show = cardNums % colNums == 0 ? ' selected' : '';
                $('.card__main__corner.--top-right',this).after('<span class="gr_nums'+show+'">'+cardNums+'</span>');
            }
            $('.card__footer__right',this).prepend('<span class="gr_delPre">&lt;&lt;&lt;&lt;&lt;</span>');
        });

        $('body').on('click','.gr_delPre',function(e){
            $(this).closest('.c-grid__item').prevAll().remove();
            e.stopPropagation();
        });

        $('body').on('click','.gr_fav',function(e){
            mid = $(this).closest('.card-model').length==1 ? $(this).closest('.card-model').data('uid') : $(this).closest('.model-smallcard').data('uid');
            url = $(this).closest('.card-model').length==1 ? $(this).closest('.card-model').find('.card-model__thumbnail-link').attr('href') : $(this).closest('.model-smallcard').find('.card-model__thumbnail-link').attr('href');
            freedl = $(this).closest('.card-model').length==1 ? $(this).closest('.card-model').find('.--downloads').length : $(this).closest('.model-smallcard').find('.--downloads').length;
            var favid = $(this).data('favid');
            $('body').find('.bootstrap-tagsinput .tag').removeClass('gr_colorbg');
            $('body').find('.bootstrap-tagsinput .tag').each(function(){
                var name = $(this).text();
                if(favType[name] == favid) {
                    $(this).addClass('gr_colorbg');
                }
            });
            $('#gr_tag').css({
                'opacity': 1,
                'position': 'fixed',
                'margin-left': 'auto',
                'left': ($(window).width()-$('#gr_tag').width())/2,
                'top': ($(window).height()-$('#gr_tag').height())/2-80
            });
            if ($('body #gr_tagbg').length == 0) $('body').prepend('<div id="gr_tagbg"> </div>');
            e.stopPropagation();
        });

        $('body').on('click', '.bootstrap-tagsinput .tag', function() {
            var type = $(this).text();
            $('body').find('.bootstrap-tagsinput .tag').removeClass('gr_tagborder gr_colorbg');
            $(this).addClass('gr_tagborder');
            if (mid) {
                var data = {"postType": "setData", "postData": {"member": _ACCOUNT, "mid": mid, "url": url, "freedl": freedl, "type": type}};
                gr_iframe.contentWindow.postMessage(data,'*');
            } else {
                alert('DOM change!');
            }
        });

        var getData = function() {
            var data = {"postType": "getData", "postData": {"member": _ACCOUNT}};
            gr_iframe.contentWindow.postMessage(data,'*');
        }
        getData();

        window.addEventListener('message',function(e){
            if (e.data.status == 1) {
                list = e.data.list;
                if (e.data.fav_type) {
                    favType = JSON.parse(e.data.fav_type);
                    $.each(favType, function(k,v){
                        $('#tagsinputval').tagsinput('add', k);
                        favList[k] = {'favid': v};
                    });
                }
                if (webSite == 'page_popular') {
                    var temp = [];
                    $.each(list, function(mid,v) {
                        var j = [mid, v.downloaded];
                        if (!temp[parseInt(v.fav_type_id)]) {
                            temp[parseInt(v.fav_type_id)] = [];
                            temp[parseInt(v.fav_type_id)][0] = [];
                            temp[parseInt(v.fav_type_id)][1] = [];
                        }
                        if (v.downloaded == 0) {
                            temp[parseInt(v.fav_type_id)][0].push(j);
                        } else {
                            temp[parseInt(v.fav_type_id)][1].push(j);
                        }
                    });
                    $.each(favList, function(k,v) {
                        favList[k].list = temp[v.favid];
                    });
                }
                if (!list) return;
                if (_disPlayMod == 1) {
                    $('.c-grid__item').each(function(){
                        var mid = $('.card-model',this).length==1 ? $('.card-model',this).data('uid') : $('.model-smallcard',this).data('uid');
                        if (list[mid]) {
                            var color = list[mid].downloaded == 0 ? 'gr_color' : 'gr_colorG';
                            $('.gr_fav',this).addClass(color).data('favid',list[mid].fav_type_id);
                        }
                    });
                } else {
                    $('.c-grid__item').each(function(){
                        var mid = $('.card-model',this).length==1 ? $('.card-model',this).data('uid') : $('.model-smallcard',this).data('uid');
                        if (list[mid]) {
                            $(this).remove();
                        }
                    });
                }
            } else if (e.data.status == 2) {
                $('body').find('#gr_tagbg').click();
                if ($('body').find('.card-model[data-uid="'+e.data.mid+'"] .gr_fav').length == 1) {
                    $('body').find('.card-model[data-uid="'+e.data.mid+'"] .gr_fav').addClass('gr_color').data('favid',e.data.fav_type_id);
                } else {
                    $('body').find('.model-smallcard[data-uid="'+e.data.mid+'"] .gr_fav').addClass('gr_color').data('favid',e.data.fav_type_id);
                }
            } else if (e.data.status == 3) {
                $('body').find('#gr_tagbg').click();
                if ($('body').find('.card-model[data-uid="'+e.data.mid+'"] .gr_fav').length == 1) {
                    $('body').find('.card-model[data-uid="'+e.data.mid+'"] .gr_fav').removeClass('gr_color gr_colorG').data('favid','');
                } else {
                    $('body').find('.model-smallcard[data-uid="'+e.data.mid+'"] .gr_fav').removeClass('gr_color gr_colorG').data('favid','');
                }
            } else if (e.data.status == 5) {
                $('body').find('#gr_tagbg').click();
                if ($('body').find('.card-model[data-uid="'+e.data.mid+'"] .gr_fav').length == 1) {
                    $('body').find('.card-model[data-uid="'+e.data.mid+'"] .gr_fav').removeClass('gr_color').addClass('gr_colorG');
                } else {
                    $('body').find('.model-smallcard[data-uid="'+e.data.mid+'"] .gr_fav').removeClass('gr_color').addClass('gr_colorG');
                }
            }

        });

        if (webSite == 'page_popular') {
            var gr_menu = '<div id="gr_menu">E Menu</div>';
            var gr_main = '<div id="gr_main">\
                <div id="gr_main_1">\
                <div id="gr_title">-- 3D模型下载列表 --</div>\
                <div id="gr_container" >\
                    <div id="gr_nav"><div class="gr_menu_fav nav_tab selected">我的收藏夹</div><div class="gr_menu_list nav_tab">导入文本列表</div></div>\
                    <div id="grdl_fav" class="grdl_tab selected">\
                        <input class="grdl_btn_fav" type="button" value="打开收藏夹">\
                    </div>\
                    <div id="grdl_list" class="grdl_tab">\
                        <div id="grdl_tab_1">\
                            <textarea name="grdl_tab_1" onfocus="document.getElementById(\'grdl_tips_1\').style.display=\'none\'" onblur="if(value==\'\')document.getElementById(\'grdl_tips_1\').style.display=\'block\'"></textarea>\
                            <div id="grdl_tips_1" class="grdl_tips">在这里粘贴网址列表，每行一条</div>\
                        </div>\
                        <div id="grdl_tab_btn">\
                            <input class="grdl_btn_list" type="button" value="导入列表">\
                        </div>\
                        <div class="grdl_tab_2">\
                            <ul></ul>\
                        </div>\
                        <div class="grdl_finished"><p>已下载历史记录：</p><ul></ul></div>\
                    </div>\
                </div>\
                </div>\
                <div id="gr_main_2"></div>\
            </div>';
            $('body h1.jS_AtdxX').append(gr_menu);
            $('body').append(gr_main);
            $('body').on('click', '#gr_menu', function() {
                $('#gr_main_1').slideDown('fast');
                $('body').prepend('<div id="gr_tagbg"> </div>');
                $('html').css('overflow-y','hidden');
                getData();
            });

            $('body').on('click', '#gr_nav .nav_tab', function() {
                $('.nav_tab, .grdl_tab').removeClass('selected');
                $(this).addClass('selected');
                if ($(this).hasClass('gr_menu_fav')) {
                    $('#grdl_fav').addClass('selected');
                    addFavList();
                } else if ($(this).hasClass('gr_menu_list')) {
                    $('#grdl_list').addClass('selected');
                }
            });
            $('body').on('click', '.grdl_btn_fav', function() {
                $(this).hide();
                addFavList();
            });

            var addFavList = function() {
                var h = window.innerHeight - 150;
                $('#grdl_fav').css({'height': h+'px'});
                $('#grdl_fav').empty();
                $.each(favList, function(k,v) {
                    var favNums;
                    if (favList[k].list) {
                        var color = favList[k].list[0].length > 0 ? ' grdl_fav_color' : '';
                        favNums = '<span class="grdl_fav_n'+color+'">'+favList[k].list[0].length+'</span>/<span class="grdl_fav_a">'+(favList[k].list[0].length+favList[k].list[1].length)+'</span>';
                    } else {
                        favNums = '<span class="grdl_fav_a">0</span>';
                    }
                    $('#grdl_fav').append('<div class="grdl_fav_tab" data-favid="'+v.favid+'"><span class="grdl_fav_tx">'+k+'</span><span class="grdl_fav_i"> '+favNums+'</span></div>');
                });
            }
            var mp1 = $('.grdl_tab_2').clone();
            var mp2 = $('.grdl_finished').clone();
            $('#gr_main_2').append(mp1);
            $('#gr_main_2').append(mp2);

            $('body').on('click', '.grdl_fav_tab', function() {
                var favid = $(this).data('favid');
                var text = $(this).find('.grdl_fav_tx').text();
                $('.grdl_fav_tab').removeClass('selected_bd');
                $(this).addClass('selected_bd');
                if (!favList[text].list) return;
                $('#gr_main_2').show().animate({right:'600px'},'fast',function() {
                    $('#gr_main_2 .grdl_tab_2 ul').empty();
                    if (favList[text].list[0].length > 0) {
                        var listNums = favList[text].list[0].length;
                        $.each(favList[text].list[0], function(k,v) {
                            var url = 'https://sketchfab.com/3d-models/'+list[v[0]].url;
                            var name = list[v[0]].url;
                            var num = listNums - k;
                            var freedlClass = list[v[0]].freedl == 1 ? 'class="freedl_color"' : '';
                            $('#gr_main_2 .grdl_tab_2 ul').append('<li><span class="grli_num">'+num+'.</span><a '+freedlClass+' href="'+url+'" target="_blank">'+name+'</a></li>');
                        });
                    }
                });
            });

            var grdl_list, grdl_listArr;
            $('body').on('click','input.grdl_btn_list',function () {
                grdl_list = $('body #grdl_tab_1 textarea').val();
                if (grdl_list == '') {
                    $('#gr_main_1 .grdl_tab_2 ul').empty();
                    return;
                }
                grdl_listArr = grdl_list.split("\n");
                grdl_listArr = grdl_listArr.filter(element => element);
                var listNums = grdl_listArr.length;
                $.each(grdl_listArr, function (k,v) {
                    if (v != "") {
                        var nameArr = v.split('/');
                        var name = nameArr.pop();
                        var num = listNums - k;
                        $('#gr_main_1 .grdl_tab_2 ul').append('<li><span class="grli_num">'+num+'.</span><a href="'+v+'" target="_blank">'+name+'</a></li>');
                    }
                });
            });

            $('body').on('click', '.grdl_tab_2 a',function () {
                $('body .grdl_finished p').remove();
                var $li = $(this).parent('li');
                if ($(this).closest('#gr_main_1').length == 1) {
                    $('#gr_main_1 .grdl_finished ul').prepend($li.clone());
                } else {
                    $('#gr_main_2 .grdl_finished ul').prepend($li.clone());
                }
                $li.remove();
            });

        }

    };


})(jQuery);
