# OpenStore – Client file upload

OpenStore can allow the client to upload files, which are then visible in the Order Admin of the Manager. (Not the client Order Admin, although the templates can be changed to allow this)  
The upload and attachment of the files to the order is done in 2 steps.  
## Step 1  
The file is selected by the client and uploaded. This is done by the use of a file input.  

### Example code require don the product detail template.
```
<script src="/DesktopModules/NBright/NBrightBuy/Themes/config/js/jquery.fileupload.js"></script>
```

```
@if (product.ClientFileUpload)
{
    <div class="options">

        <div id="clientfileuploadmsg">
            @ResourceKey("ProductView.clientfileuploadmsg")
        </div>
        <div id="clientfileuploadmsgend" style="display: none;">
            @ResourceKey("ProductView.clientfileuploadmsgend")
        </div>
        <div class="fileUpload">
            <input id='cmdClientUpload' update='save' class="normalFileUpload form-control" type='file' name="files[]" multiple />
        </div>
        <div id="progressbar" style="display: none;">
            <div id="progressbarpercent"></div>
        </div>

		<input id='clientuploadfilelist' style='display:none;' update='save' type='text' />

    </div>
    <div class="clearfix"></div>
}

```

When the files are selected a trigger is made to upload the files to the \Portals\#\NBSTemp folder and to populate a input field called “clientuploadfilelist”. 

```
function initClientFileUpload() {
	$('#clientuploadfilelist').val('');
    var filecount2 = 0;
    var filesdone2 = 0;
    $(function () {
        'use strict';
        var url = '/DesktopModules/NBright/NBrightBuy/XmlConnector.ashx?cmd=fileclientupload';
        $('#cmdClientUpload').unbind();
        $('#cmdClientUpload').fileupload({
            url: url,
            maxFileSize: 5000000,
            acceptFileTypes: /(\.|\/)(jpg|png|jpeg|pdf)$/i,
            dataType: 'json'
        }).prop('disabled', !$.support.fileInput).parent().addClass($.support.fileInput ? undefined : 'disabled')
            .bind('fileuploadprogressall', function (e, data) {
                var progress = parseInt(data.loaded / data.total * 100, 10);
                $('#progress .progress-bar').css('width', progress + '%');
            })
            .bind('fileuploadadd', function (e, data) {
                $.each(data.files, function (index, file) {
                    $('#clientuploadfilelist').val($('#clientuploadfilelist').val() + file.name + ',');
                    filesdone2 = filesdone2 + 1;
                });
            })
            .bind('fileuploadchange', function (e, data) {
                filecount2 = data.files.length;
                $('.processing').show();
            })
            .bind('fileuploaddrop', function (e, data) {
                filecount2 = data.files.length;
                $('.processing').show();
            })
            .bind('fileuploadstop', function (e) {
                if (filesdone2 == filecount2) {
                    filesdone2 = 0;
                    $('.processing').hide();
                    $('#progress .progress-bar').css('width', '0');
					$('#clientfileuploadmsgend').show();
					$('#clientfileuploadmsg').hide();
					$('#cmdClientUpload').hide();					
                }
            });
    });

}
```

```
$(document).ready(function () {
    initClientFileUpload();
});
```

On upload the filename is encrypted using the store admin pin as a passkey.  This is so the filename on the server is not known to a potential hacker.  
 
## Step 2
When the item is added to the basket, the “clientuploadfilelist” field is used to process the uploaded files and attach them to the cart.

### NOTES:
Client Files are moved from the temp folder to the “clientupload” folder, when added to the basket and are renamed to a random key name, so the files cannot be recognised.  
The API command for client file upload is : “fileclientupload”  
The option in store settings for allowing client upload must be set for uploads to work.  
There is an option in the product admin to activate the product client file upload option.  
There is no code on the “ClassicAjax” default templates to make uploading client files possible.  You need the above code added to the Product detail template. 

**NOTE: Although precautions have been made to protect the upload from hackers, allowing uploading of files to your server can be VERY dangerous.  Be aware of how to secure your server from external access.**