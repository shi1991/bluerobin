<template>
    <div class="task-textfield">
      <input class="task-textfield__input" type="text" v-model="newTask.title" @keyup.enter="createTask(list_id)">
    </div>
</template>

<script>
import Uploader from '../../services/upload.babel.js';
import Tool from '../../services/tool.babel.js';

import * as taskActions from '../../vuex/actions/tasks';
import * as getters from '../../vuex/getter';

function guid() {
  function s4() {
    return Math.floor((1 + Math.random()) * 0x10000)
      .toString(16)
      .substring(1);
  }
  return s4() + s4() + '-' + s4() + '-' + s4() + '-' +
    s4() + '-' + s4() + s4() + s4();
}

export default {
  data() {
    return {
      list_id: '',
      newTask: {
        title: '',
        list_id: '',
        create_time: '',
        attachments: []
      },
      imagePreviewList: [],
      uploader: null,
      newFile: null,
      watchData: []
    };
  },
  vuex: {
    actions: {
      addTask: taskActions.addTask,
    },
  },
  filters: {
    blob2src: function(blob) {
      return URL.createObjectURL(blob);
    }
  },
  watch: {
    'newTask.title': function(val, oldval) {
      localStorage.newTask = JSON.stringify(this.newTask);
    },
    'newTask.attachments': function(val, oldval) {
      localStorage.newTask = JSON.stringify(this.newTask);
    }
  },
  ready() {
    this.list_id = this.$route.params.id;
    this.watchData = [this.newTask.title, this.newTask.attachments];

    this.newTask = window.localStorage.newTask ? JSON.parse(localStorage.newTask) : {};
    componentHandler.upgradeDom();

    this.init();
  },

  methods: {
    // 初始化
    init() {
      let _this = this;
      this.$set('uploader', Uploader({
        container: 'taskWriter'
      }));

      this.uploader.bind('PostInit', function() {
        _this.uploader.addFile(_this.newFile);
      });

      this.uploader.bind('BeforeUpload', function(up, file) {

      });

      this.uploader.bind('FileUploaded', function(up, file, res) {
        _this.newTask.attachments.push({
          name: file.name,
          url: Tool.uploadImageSrc(file.name),
          size: file.size,
          width: file.width,
          height: file.height,
          type: file.type
        });
      });
    },

    // 创建任务
    createTask(listid) {
      if (!this.newTask.title) {
        return false;
      }

      this.$set('newTask.create_time', new Date());
      this.$set('newTask.list_id', listid);

      this.addTask(this.newTask);

      this.$set('newTask', {
        title: '',
        create_time: '',
        attachments: []
      });
    },

    // 粘贴复制
    uploadByPaste(e) {
      var _this = this;
      var items = e.clipboardData && e.clipboardData.items;
      if (!(items && items.length)) {
        return false;
      }
      for (var i = 0; i < items.length; i++) {
        var file = items[i].getAsFile && items[i].getAsFile();
        if (file) {
          file.name = guid() + '.' + file.type.replace(/^\w*\//ig, '');
          _this.uploader.addFile(file);
          _this.newFile = file;
        }
      }
    }
  }

};

</script>



<style lang="less">

@import '../../public/stylesheets/variables';

.modal.bottom-sheet{
  max-height: 60%;
}

.task-textfield{
  width: 100%;
  background: #fff;
  margin-bottom: 20px;
  box-sizing: border-box;
  &__input{
    width: 100%;
    height: 50px;
    font-size: 20px;
    padding: 6px 10px;
    box-sizing: border-box;
    border: none;
    outline: none;
    &:focus{
      box-shadow: 0 0 5px #ABABAB;
    }
  }
}

</style>
