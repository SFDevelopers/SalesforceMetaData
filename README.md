SalesforceMetaData
==================


<apex:page controller="SFMetaData" sidebar="false" showHeader="false" docType="html-5.0" standardStylesheets="false" cache="false">
<head>
<style>
    ul {white-space: nowrap;}
    .twt-border {
        -webkit-box-shadow: 0 1px 3px rgba(0,0,0.15)!important;
        -moz-box-shadow: 0 1px 3px rgba(0,0,0.15)!important;
        box-shadow: 0 1px 3px rgba(0,0,0.15)!important;
        border: #ddd 1px solid!important;
        border-top-color: #eee!important;
        border-bottom-color: #bbb!important;
        background-color: #fff!important;
        padding:20px;
        border-bottom-right-radius: 6px;
        border-bottom-left-radius: 6px;
        width:390px;
        align:center;
    }
    ul{list-style: none; padding:5px;}
    ul li {padding-bottom:5px;}
    .SObject_Custom_true_CusSetting_false { color: #01A; }
    .SObject_Custom_true_CusSetting_true { color: #0C0; }
   .inlineDisplay{display:inline-block;  vertical-align: top;}
   //    .SObject_Custom_false_CusSetting_false { color: #0AF; }
</style>

<apex:includeScript value="{!$Resource.jquery_min_V172}"/>
<apex:includeScript value="{!$Resource.jQueryTemplate}"/>
</head>
<body onmousemove="$(function(){ $('.s').height($(window).height()-150);});" style="overflow:auto;">
<div>
<ul id="Container">
<li class="inlineDisplay">
    <ul>
        <li  style="background:whitesmoke; border-top-left-radius: 10px; border-top-right-radius: 10px; padding: 8px; padding-left: 20px; padding-right:20px; border:1px solid #bbb; width:390px; align:center;">
            <div style="text-align:center; padding-bottom:3px;"><b>SObjects</b></div>
            <div>
                <input type="checkbox" name="Standard" value="Standard" checked="checked" onchange="$('.SObject_Custom_false_CusSetting_false').toggle();" />Standard &nbsp;&nbsp;
                <input type="checkbox" name="Custom" value="Custom" checked="checked" onchange="$('.SObject_Custom_true_CusSetting_false').toggle();" />Custom &nbsp;&nbsp;
                <input type="checkbox" name="CustomSetting" value="CustomSetting" checked="checked" onchange="$('.SObject_Custom_true_CusSetting_true').toggle();" />Custom Setting
            </div>
        </li>
        <li>
            <div  class="twt-border">
                <div id="sObjectList"  class="s" style="overflow:auto;"/>
            </div>
        </li>
    </ul>
</li>
</ul>
</div>
</body>


<script>
    $(".s").height($(window).height()-200);
    
    $("body").height($(window).height());
    window.sessionStorage.removeItem('alreadyFetchedObj');
        
        var params = {};
        var queryString = location.hash.substring(1);
        var re = /([^&=]+)=([^&]*)/g;
        var m;
        while( m = re.exec(queryString) ) {
          params[decodeURIComponent(m[1])] = decodeURIComponent(m[2]);
        }
        var c = JSON.stringify(params).toString();
        if(c!= null ) {
            SFMetaData.callRequest(c,'/services/data/v24.0/sobjects/',function(result){ $temp_res = JSON.parse(result);
                                                        console.debug($temp_res);
                                                        $("#sfObjectsList").tmpl($temp_res).appendTo("#sObjectList");
                                                        $temp_res = '';
                                                     },{escape:false});
        }
</script>  
<script id="sfObjectsList" type="text/x-jquery-tmpl"> 
    <ul id="SObjects" style="list-style-type:circle;">
            {{each sobjects}}
                <li class="SObject_Custom_${custom}_CusSetting_${customSetting}" title="label : ${label}" onclick="fetchFieldDetais('${name}','${urls.describe}');">${name}</li>
            {{/each}}
    </ul>
</script>
<script>
    var $alredayFetchObj=$temp1=$temp_res=$localStorageString='';
    
    function fetchFieldDetais(name,describe){
        if(window.sessionStorage.getItem('alreadyFetchedObj') == null){window.sessionStorage.setItem('alreadyFetchedObj', $localStorageString+',s'+name+'s,'); serverReq(name,describe);}
        else{
                $localStorageString = window.sessionStorage.getItem('alreadyFetchedObj');
                if($localStorageString.indexOf(',s'+name+'s,') >= 0){$("#ObjectFieldDetalsFor_"+name).insertAfter($("#Container li:first"));}
                else{window.sessionStorage.setItem('alreadyFetchedObj', $localStorageString+',s'+name+'s,'); serverReq(name,describe);}
        }
    }
    function serverReq(name,describe)
    {
        SFMetaData.callRequest(c,describe,function(result){ 
                                                        $temp_res = JSON.parse(result);
                                                        console.debug($temp_res);
                                                        $("#sfObjectFieldsDetails").tmpl($temp_res).insertAfter($("#Container li:first"));
                                                        $temp_res = '';
                                                        var h1 = $('#Container').children('li').length;
                                                        var h2 = $(window).width();
                                                        var t = ((36000*h1)/(h2));  at = t+'%';  $('body').width(at);
                                                    },{escape:false});  
    }
</script>   
<script id="sfObjectFieldsDetails" type="text/x-jquery-tmpl">

<li  class="inlineDisplay" id="ObjectFieldDetalsFor_${name}">
    <ul>
        <li  style="background:whitesmoke; border-top-left-radius: 10px; border-top-right-radius: 10px; padding: 8px; padding-left: 20px; padding-right:20px; border:1px solid #bbb; width:390px; align:center;">
            <div style="text-align:center; padding-bottom:3px;"><b>${name}</b></div>
            <ul style="padding: 0px;"><li class="inlineDisplay" style="padding:0px;">Fields : ${fields.length}</li><li class="inlineDisplay" onclick="scrollToChildRelation(this);" style="padding:0px; float:right;">ChildRelationships : ${childRelationships.length}</li></ul>
        </li>
        <li>
            <div  class="twt-border">
                <div id="sObjectFieldsDetails"  class="s" style="overflow:auto;">
                    
                    
                    <ul id="SObjectFields" style="list-style-type:square;">
                        {{each fields}}
                            <li class="SFields_Custom_${custom}" id="${name}" title="label : ${label}">                        <!--//$('#${name}').prop('checked', !$('#${name}')[0].checked);-->
                                <a href="javascript:void(0)"  onclick="expandTree(this);" class="plusArrow">&nbsp;&nbsp;&nbsp;</a>
                                <input type="checkbox" name="FieldSel" value="FieldSel" id="${name}" /><span {{if custom}}class="SObject_Custom_true_CusSetting_false"{{/if}}>${name}</span>
                                <div style="float:right; width:60px; font-size:small; text-align:left;">${type}</div >
                                <ul style="margin-left:20px; font-size:small; background:#FFE; display:none">
                                    {{if relationshipName}}<li>RelationshipName : ${relationshipName}</li>{{/if}}
                                    {{if autoNumber}}<li>AutoNumber</li>{{/if}}
                                    {{if unique}}<li>Unique</li>{{/if}}
                                    {{if externalId}}<li>ExternalId</li>{{/if}}
                                    {{if customField}}<li>CustomField</li>{{/if}}
                                    <li>
                                        <a href="javascript:void(0)"  onclick="expandTree(this);" class="plusArrow">&nbsp;&nbsp;&nbsp;</a>
                                        Access
                                        <ul style="margin-left:20px; font-size:small; background:lightyellow; display:none"  class="pickListAlt">
                                            {{if createable}}<li>Createable</li>{{/if}}
                                            {{if updateable}}<li>Updateable</li>{{/if}}
                                            {{if nillable}}<li>Nillable</li>{{/if}}
                                            {{if filterable}}<li>Filterable</li>{{/if}}
                                            {{if defaultedOnCreate}}<li>DefaultedOnCreate</li>{{/if}}
                                            {{if deprecatedAndHidden}}<li>DeprecatedAndHidden</li>{{/if}}
                                            {{if writeRequiresMasterRead}}<li>WriteRequiresMasterRead</li>{{/if}}
                                            {{if permissionable}}<li>Permissionable</li>{{/if}}
                                            {{if restrictedDelete}}<li>RestrictedDelete</li>{{/if}}
                                        </ul>
                                    </li>
                                    <li>
                                        <a href="javascript:void(0)"  onclick="expandTree(this);" class="plusArrow">&nbsp;&nbsp;&nbsp;</a>
                                        DataType - ${type}
                                        <ul style="margin-left:20px; font-size:small; background:lightyellow; display:none"  class="pickListAlt">
                                            {{if htmlFormatted}}<li>Field is HTML Formated (Rich Text AreaType).</li>{{/if}}
                                            {{if type=='currency'||type=='double'||type=='Percent'}}
                                                <li>Precision : ${precision}</li>
                                                <li>Scale : ${scale}</li>
                                            {{/if}}
                                            {{if type=='string'||type=='textarea'||type=='reference'||type=='phone'||type=='reference'||type=='id'||type=='picklist'||type=='url'||type=='email'}}
                                                <li>Length : ${length}</li>
                                                <li>ByteLength : ${byteLength}</li>
                                            {{/if}}
                                            <li>SoapType : ${soapType}</li>
                                        </ul>
                                    </li>
                                    {{if type=='picklist'}}
                                        <li>
                                            <a href="javascript:void(0)"  onclick="expandTree(this);" class="plusArrow">&nbsp;&nbsp;&nbsp;</a>
                                            picklistValues
                                            <ul style="margin-left:20px; font-size:small; background:lightyellow; display:none"  class="pickListAlt">
                                                {{each picklistValues}}
                                                    <li>
                                                        <a href="javascript:void(0)"  onclick="expandTree(this);" class="plusArrow">&nbsp;&nbsp;&nbsp;</a>${label}
                                                        <ul style="margin-left:20px; font-size:small; background:#ffe; display:none">
                                                            <li>Label : ${label}</li>
                                                            <li>Value : ${value}</li>
                                                            <li>isActive : ${active}</li>
                                                            <li>isDefault : ${defaultValue}</li>
                                                        </ul>
                                                    </li>
                                                {{/each}}
                                            </ul>
                                         </li>
                                    {{/if}}
                                    {{if type=='reference'}}
                                        <li>
                                            <div style="maxwidth:252px; overflow:auto;">
                                                <a href="javascript:void(0)"  onclick="expandTree(this);" class="plusArrow">&nbsp;&nbsp;&nbsp;</a>
                                                Reference To
                                                <ul style="margin-left:20px; font-size:small; display:none" >
                                                    {{each referenceTo}}
                                                        <li>
                                                            <a href="javascript:void(0)"  onclick="expandTree2(this,'${$value}');" class="plusArrow">&nbsp;&nbsp;&nbsp;</a>${$value}
                                                            <ul style="margin-left:20px; font-size:small; display:none" />
                                                        </li>
                                                    {{/each}}
                                                </ul>
                                            </div>
                                         </li>
                                    {{/if}}
                                </ul>
                            </li>
                        {{/each}}
                    </ul>
                    {{if childRelationships!=0}}
                    <ul id="ChildSObjectFieldsOneLevel">
                        <li>
                            <a href="javascript:void(0)"  onclick="expandTree(this);" class="plusArrow">&nbsp;&nbsp;&nbsp;</a>Child Relationships
                            <ul style="margin-left:10px; font-size:medium; background:whitesmoke; display:none">
                                {{each childRelationships}}
                                    {{if relationshipName}}
                                        <li>
                                            <a href="javascript:void(0)" onclick="expandTree2(this,'${childSObject}');" class="plusArrow">&nbsp;&nbsp;&nbsp;</a>${childSObject}
                                            <ul style="margin-left:10px; font-size:small; background:lightyellow; display:none"  class="pickListAlt">
                                                <li>&nbsp;&nbsp;RelationName : ${relationshipName}</li>
                                                <li>&nbsp;&nbsp;Related Field : ${field}</li>
                                                {{if cascadeDelete}}<li>&nbsp;&nbsp;Cascade Delete</li>{{/if}}
                                                {{if deprecatedAndHidden}}<li>&nbsp;&nbsp;Deprecated And Hidden</li>{{/if}}
                                                {{if restrictedDelete}}<li>&nbsp;&nbsp;Restricted Delete</li>{{/if}}
                                            </ul>
                                        </li>
                                    {{else}}
                                        <li><a href="javascript:void(0)" class="noArrow">&nbsp;&nbsp;&nbsp;</a>${childSObject}</li>
                                    {{/if}}
                                {{/each}}
                            </ul>
                        </li>
                    </ul>
                    {{/if}}
                
                </div>
            </div>
        </li>
    </ul>
</li>    
</script>   
<script>
    function scrollToChildRelation(tempObj)
    {
        $(tempObj).parent().parent().parent().find("#sObjectFieldsDetails").scrollTop(99999999);
    }
    function expandTree(tempObj) {
        $(tempObj).toggleClass('minusArrow'); 
        $(tempObj).parent().children('ul').toggle();
    }
    function Child_Sel_All(tempChildSelAllObj) {
        
        if($(tempChildSelAllObj).attr("checked"))
            $(tempChildSelAllObj).siblings("ul").find("input").each(function(){$(this).attr("checked","checked");});
        else
            $(tempChildSelAllObj).siblings("ul").find("input").each(function(){$(this).removeAttr("checked");});
    }
    function expandTree2(tempObj,childObj) {
        expandTree(tempObj);
        SFMetaData.callRequest(c,'/services/data/v24.0/sobjects/'+childObj+'/describe',function(result){ 
                                                        $temp_res = JSON.parse(result);
                                                        console.debug($temp_res);
                                                        console.debug(tempObj);
                                                        $("#sfChildObjectFieldsDetails").tmpl($temp_res).appendTo($(tempObj).siblings('ul'));
                                                        $('<a href="javascript:void(0)" onclick="expandTree(this);" class="plusArrow minusArrow">&nbsp;&nbsp;&nbsp;</a>').replaceAll(tempObj);
                                                        $temp_res = '';
                                                    },{escape:false});
    }
</script>

<script id="sfChildObjectFieldsDetails" type="text/x-jquery-tmpl">
    <li>
        <a href="javascript:void(0)"  onclick="expandTree(this);" class="plusArrow minusArrow">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</a>
        <input type="checkbox" name="ChildFieldSelAll" value="ChildFieldSelAll" id="Child_Sel_All" onclick="Child_Sel_All(this);"/>&nbsp;&nbsp;Fields - ${fields.length}
        <ul style="margin-left:15px; font-size:medium; background:whitesmoke;" id="Child_childRelationships">
            {{each fields}}<li><input type="checkbox" name="ChildFieldSel" value="ChildFieldSel" id="Child_${name}" />${name}</li>{{/each}}
        </ul>
    </li>
</script>


<style>
    ::-webkit-scrollbar {
        width: 8px;
        height: 15px;
    }
    ::-webkit-scrollbar-button:start:decrement,::-webkit-scrollbar-button:end:increment {
        height: 1px;
        display: block;
        background-color: #EFF7FF;
    }
    ::-webkit-scrollbar-button:horizontal:start:decrement,::-webkit-scrollbar-button:horizontal:end:increment {
        height: 1px;
        display: block;
        background-color: #EFF7FF;
    }
    ::-webkit-scrollbar-track-piece {
        background-color: #EFF7FF;
    }
    ::-webkit-scrollbar-thumb:vertical,::-webkit-scrollbar-thumb:horizontal {
        background-color: white;
        border: 1px solid #639ACE;
        -webkit-border-radius: 6px;
    }
    a{text-decoration:none;}
    td{vertical-align:top;}
    .s li:hover{background:whitesmoke;}
    .plusArrow{
        background:url(/resource/1336044224000/S_MetaDataImages/plus_arrow.gif) no-repeat;
        background-position: 1px 3px;
        width: 165px;
        overflow: hidden;
    }
    .minusArrow{
        background:url(/resource/1336044224000/S_MetaDataImages/minus_Arrow.gif) no-repeat;
        background-position: 1px 3px;
        width: 165px;
        overflow: hidden;
    }
    .noArrow{
        background:url(/resource/1336044224000/S_MetaDataImages/gray_Dot.png) no-repeat;
        background-position: 1px 6px;
        width: 165px;
        overflow: hidden;
    }
    .closeButton{
        background: transparent url(http://www.poptrickia.net/files/theme/close-button.png) no-repeat;
        display:inline; width:40px;height:40px;z-index:100;right:448px;top:7px;
    }
    
//    .pickListAlt>li:nth-child(odd){background:lightyellow;}
</style>    
</apex:page>