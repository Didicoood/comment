<template>
  <div>
    <Comment>
      <!-- <template #avatar>
        <Avatar :src="props.userInfo.avatar" alt="Han Solo" />
      </template> -->
      <template #content>
        <FormItem>
          <a-textarea :rows="4" @change="handleChange" v-model:value="value" />
        </FormItem>
        <FormItem>
          <a-button htmlType="submit" :loading="submitting" @click="handleSubmit" type="primary">
            发表评论
          </a-button>
        </FormItem>
      </template>
    </Comment>
  </div>
</template>
<script setup>
  import { ref } from 'vue';
  import { Comment, Form } from 'ant-design-vue';
  import $api from '../api';
  import { inject } from 'vue';

  const getCommentList = inject('getCommentList');
  const FormItem = Form.Item;
  // const appStore = useAppStore();
  const props = defineProps({
    userInfo: Object,
    replyInfo: Object,
    showCancle: {
      type: Boolean,
      default: true,
    },
    guideId: String,
    repliedId: String,
  });

  const commentContent = ref();
  const submitting = ref(false);
  const comments = ref([]);
  const value = ref('');
  const handleSubmit = async () => {
    const newComment = {
      content: commentContent.value,
      guideId: props.guideId,
      parentId: props.repliedId,
    };
    // 去除undefined的属性
    for (let key in newComment) {
      if (newComment[key] === undefined) {
        delete newComment[key];
      }
    }
    console.log(commentContent.value, props.guideId, '我是valuecomment');
    console.log(props.repliedId, '我是props.parentId');
    const res = await $api.saveComment(newComment);
    console.log(res, '我是柏翠和回复');
    comments.value.push(newComment);
    commentContent.value = '';
    let lists = document.getElementsByClassName('comment');
    for (let i = 0; i < lists.length; i++) {
      lists[i].style.display = 'none';
    }
    getCommentList();
    value.value = '';
  };

  const handleChange = (e) => {
    commentContent.value = e.target.value;
  };
</script>

<style lang="less" scoped>
  :deep .ant-input {
    margin-bottom: -10px;
  }
</style>
