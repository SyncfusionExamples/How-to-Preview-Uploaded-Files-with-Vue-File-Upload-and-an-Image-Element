<template>
  <div class="container">
    <div id="dropArea" class="uploader-image-preview-drop-area">
      <span>
        Drop image (JPG, PNG) files here or <a href="#" id="browse"><u>Browse</u></a>
      </span>
      <ejs-uploader ref="uploaderObj" :asyncSettings="asyncSettings" :allowedExtensions="'.jpg, .png'"
        :dropArea="dropElement" cssClass="uploader-preview"
        :selected="onFileSelected"
        :success="onSuccess"
        ></ejs-uploader>
    </div>
  </div>
</template>

<script>
import { UploaderComponent } from '@syncfusion/ej2-vue-inputs';
import { createElement, isNullOrUndefined, detach } from '@syncfusion/ej2-base';

export default {
  components: {
    'ejs-uploader': UploaderComponent
  },
  data() {
    return {
      asyncSettings: {
        saveUrl: 'https://services.syncfusion.com/vue/production/api/FileUploader/Save',
        removeUrl: 'https://services.syncfusion.com/vue/production/api/FileUploader/Remove'
      },
      dropElement: '#dropArea',
      fileElements: [],
      filesDetails: []
    };
  },
  mounted() {
    document.getElementById('browse').onclick = function () {
      document.querySelector('.e-file-select-wrap').querySelector('button').click();
      return false;
    };
  },
  methods:{
    onFileSelected(selectedEvent){
      if(isNullOrUndefined(document.getElementById('dropArea').querySelector('.e-upload-files'))){
      document.getElementsByClassName('uploader-preview')[0].appendChild(createElement('ul',{className:'e-upload-files'}));
      }
      selectedEvent.filesData.forEach(file=>{
        this.formSelectedData(file);
      });
      this.filesDetails = [...this.filesDetails, ...selectedEvent.filesData];
      selectedEvent.cancel = true;
    },
    formSelectedData(file){
      let liEle = createElement('li', {className: 'e-upload-file-list', attrs: {'data-file-name': file.name }});
      let imageElement = createElement('img', { className: 'upload-image'});
      let wrapper = createElement('span');
      wrapper.appendChild(imageElement);
      liEle.appendChild(wrapper);
      liEle.appendChild(createElement('div', {className:'file-name', innerHTML: file.name }));
      liEle.appendChild(createElement('div', {className:'file-size', innerHTML:
        this.$refs.uploaderObj.bytesToSize(file.size)
      }));
      let uploadBtn = createElement('span', { id: 'iconUpload', className:
        'e-upload-icon e-icons e-file-remove-btn'
      });
      uploadBtn.addEventListener('click', this.uploadFile);
      liEle.appendChild(uploadBtn);

      let removeBtn = createElement('span', { id: 'removeIcon', className: 
      'e-icons e-file-remove-btn'
      });
      removeBtn.addEventListener('click', this.removeFile);
      liEle.appendChild(removeBtn);

      let reader = new FileReader();
      reader.onload = () => {
        liEle.querySelector('.upload-image').src = reader.result;
      };
      reader.readAsDataURL(file.rawFile);

      document.querySelector('.e-upload-files').appendChild(liEle);
      this.fileElements.push(liEle);
    },
    uploadFile(event){
      let fileIndex = this.fileElements.indexOf(event.currentTarget.parentElement);
      this.$refs.uploaderObj.upload([this.filesDetails[fileIndex]], true);
    },
    onSuccess(event){
      let li = document.querySelector(`[data-file-name="${event.file.name}"]`);
      if(event.operation === 'upload'){
        li.querySelector('#iconUpload').removeEventListener('click', this.uploadFile);
        li.querySelector('.file-name').style.color = '#4CAF50';
        li.querySelector('#removeIcon').classList.replace('e-file-remove-btn','e-file-delete-btn');
      }else if(event.operation === 'remove'){
        this.filesDetails.splice(this.fileElements.indexOf(li),1);
        this.fileElements.splice(this.fileElements.indexOf(li),1);
        detach(li);
      }
    },
    removeFile(event){
      let fileIndex = this.fileElements.indexOf(event.currentTarget.parentElement);
      let fileInfo = this.filesDetails[fileIndex];
      if(fileInfo.statusCode === '2'){
        this.$refs.uploaderObj.remove(fileInfo);
        this.$refs.uploaderObj.clearAll();
      }else if(fileInfo.statusCode === '1' || fileInfo.statusCode === '0'){
        this.filesDetails.splice(fileIndex,1);
        this.fileElements.splice(fileIndex,1);
        detach(event.currentTarget.parentElement);
        this.$refs.uploaderObj.clearAll();
      }
    }
  }
};
</script>

<style>
@import "../node_modules/@syncfusion/ej2-base/styles/material.css";
@import "../node_modules/@syncfusion/ej2-buttons/styles/material.css";
@import "../node_modules/@syncfusion/ej2-vue-inputs/styles/material.css";

@font-face {
  font-family: 'Uploader_Icon';
  src:
    url(data:application/x-font-ttf;charset=utf-8;base64,AAEAAAAKAIAAAwAgT1MvMj1tSfIAAAEoAAAAVmNtYXDnEOdVAAABiAAAADZnbHlmoZcPvgAAAcgAAABAaGVhZBLQTSUAAADQAAAANmhoZWEINQQDAAAArAAAACRobXR4CAAAAAAAAYAAAAAIbG9jYQAgAAAAAAHAAAAABm1heHABDgAdAAABCAAAACBuYW1lQySinQAAAggAAAIxcG9zdLfl0usAAAQ8AAAAMgABAAAEAAAAAFwEAAAAAAAD2AABAAAAAAAAAAAAAAAAAAAAAgABAAAAAQAA2vKJUF8PPPUACwQAAAAAANftBBgAAAAA1+0EGAAAAAAD2AP4AAAACAACAAAAAAAAAAEAAAACABEAAgAAAAAAAgAAAAoACgAAAP8AAAAAAAAAAQQAAZAABQAAAokCzAAAAI8CiQLMAAAB6wAyAQgAAAIABQMAAAAAAAAAAAAAAAAAAAAAAAAAAAAAUGZFZABA5wDnAAQAAAAAXAQAAAAAAAABAAAAAAAABAAAAAQAAAAAAAACAAAAAwAAABQAAwABAAAAFAAEACIAAAAEAAQAAQAA5wD//wAA5wD//wAAAAEABAAAAAEAAAAAAAAAIAAAAAIAAAAAA9gD+AAHABAAADchESMVITUjEzM3ETMRFzMBKAOwPvzMPp1mtUC1Zv7FCAF6vb0BO7X+EAHwtQE7AAAAABIA3gABAAAAAAAAAAEAAAABAAAAAAABAAgAAQABAAAAAAACAAcACQABAAAAAAADAAgAEAABAAAAAAAEAAgAGAABAAAAAAAFAAsAIAABAAAAAAAGAAgAKwABAAAAAAAKACwAMwABAAAAAAALABIAXwADAAEECQAAAAIAcQADAAEECQABABAAcwADAAEECQACAA4AgwADAAEECQADABAAkQADAAEECQAEABAAoQADAAEECQAFABYAsQADAAEECQAGABAAxwADAAEECQAKAFgA1wADAAEECQALACQBLyBVcGxvYWRlclJlZ3VsYXJVcGxvYWRlclVwbG9hZGVyVmVyc2lvbiAxLjBVcGxvYWRlckZvbnQgZ2VuZXJhdGVkIHVzaW5nIFN5bmNmdXNpb24gTWV0cm8gU3R1ZGlvd3d3LnN5bmNmdXNpb24uY29tACAAVQBwAGwAbwBhAGQAZQByAFIAZQBnAHUAbABhAHIAVQBwAGwAbwBhAGQAZQByAFUAcABsAG8AYQBkAGUAcgBWAGUAcgBzAGkAbwBuACAAMQAuADAAVQBwAGwAbwBhAGQAZQByAEYAbwBuAHQAIABnAGUAbgBlAHIAYQB0AGUAZAAgAHUAcwBpAG4AZwAgAFMAeQBuAGMAZgB1AHMAaQBvAG4AIABNAGUAdAByAG8AIABTAHQAdQBkAGkAbwB3AHcAdwAuAHMAeQBuAGMAZgB1AHMAaQBvAG4ALgBjAG8AbQAAAAACAAAAAAAAAAoAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAIBAgEDAAhVcGxvYWRlcgAAAAA=) format('truetype');
  font-weight: normal;
  font-style: normal;
}

.uploader-preview.e-upload .e-upload-files .e-icons.e-upload-icon {
  font-family: 'Uploader_Icon';
  font-size: 15px;
  left: 20px;
}

.uploader-preview .e-upload-files .e-file-delete-btn.e-icons,
.uploader-preview .e-upload-files .e-file-remove-btn.e-icons {
  top: 120px;
  background-color: rgb(246, 245, 245);
  border-radius: 50%;
  font-size: 12px;
  left: 80px;
}

.uploader-preview .e-upload-files li .e-icons {
  visibility: hidden;
}

.uploader-preview .e-upload-files li:hover .e-icons {
  visibility: visible;
}

.uploader-preview.e-upload .e-upload-files .e-icons:not(.e-uploaded):hover {
  background-color: #e6e6e6;
  border-color: #adadad;
  color: #333;
}

.uploader-preview.e-upload .e-upload-files .e-icons.e-upload-icon::before {
  content: '\e700';
}

.uploader-preview .file-name {
  padding: 3px 10px;
  overflow: hidden;
  text-overflow: ellipsis;
  width: 90%;
  white-space: nowrap;
}

.uploader-preview .file-size {
  padding: 0px 10px;
}

.uploader-preview .upload-image{
  height: 150px;
  width: 150px;
  margin: 10px;
}
.container {
  position: absolute;
  top: 15%;
  left: 40%;
  max-width: 505px;
  margin: auto;
}

.uploader-image-preview-drop-area { 
  border: 1px dashed #c3c3cc;
  position: relative;
  text-align: center;
  padding: 20px 5px 18px 5px;
}

.uploader-preview.e-upload .e-upload-files {
  text-align: initial;
  border-top: none;
}

.uploader-preview .e-file-select-wrap {
  display: none;
}

.e-upload {
  border: 0px;
}

.e-upload .e-upload-files .e-upload-file-list {
  border: 0px;
  display: inline-block;
  width: 165px;
} 

</style>
