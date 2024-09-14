<template>
  <div>
    <Comment v-for="item in props.comments" :key="item.uid">
      <template #actions>
        <span key="comment-basic-like">
          <a-tooltip title="Like">
            <template v-if="item.isStar === 1">
              <LikeFilled @click="like(item.id)" class="activeLike" />
            </template>
            <template v-else>
              <LikeOutlined @click="like(item.id)" />
            </template>
          </a-tooltip>
          <span style="padding-left: 8px; cursor: auto">
            {{ item.starNum }}
          </span>
        </span>
        <span>{{ dayjs(item.createTime).format('YYYY-MM-DD HH:mm:ss') }}</span>
        <span
          key="comment-basic-delete"
          @click="deleteComment(item.id)"
          v-if="props.del == 1 || item.userId == props.currentUserId"
          >删除
        </span>
        <span
          key="comment-basic-reply-to"
          @click="replyTo(item.id)"
          v-if="item.parentId === null && props.reply == 1 && item.reply === null"
          >{{ !item.commnetFlag ? '回复' : '取消回复' }}</span
        >
      </template>
      <template #author
        ><a>{{ item.userName }}</a>
        <img v-if="props.reply === 1" :src="BUCKET_OMS_IMAGES + '/help/administrator.png'" alt=""
      /></template>
      <template #avatar>
        <Avatar
          :src="herfLocation + '/system/user/userProfileViewImg?userId=' + item.userId"
          :alt="item.userName"
        />
      </template>
      <template #content>
        <p :class="{ commentContentColor: item.parentId }" v-if="item.parentId">
          {{ item.content }}{{ `@` + props.repliedUserName }}
        </p>
        <p :class="{ commentContentColor: item.parentId }" v-else>
          {{ item.content }}
        </p>
      </template>

      <CommentBox
        class="comment"
        :id="item.id"
        :guideId="guideId"
        :repliedId="repliedId"
        v-if="!item.reply"
      />
      <!-- <CommentBox
        class="comment"
        :userInfo="userInfo"
        :reply-info="replyInfo"
        :id="item.id"
        :guideId="guideId"
        :repliedId="repliedId"
        v-if="!item.reply"
      /> -->
      <!-- 二级评论,需要再传数据进去才能对二级评论生效 -->
      <CommentList
        :comments="item.reply"
        :reply="props.reply"
        :del="props.del"
        :currentUserId="props.currentUserId"
        :repliedUserName="item.userName"
      />
    </Comment>
  </div>
</template>
<script>
  export default {
    name: 'CommentList',
  };
</script>
<script setup>
  import CommentBox from './CommentBox.vue';
  import { Comment, Avatar } from 'ant-design-vue';
  import dayjs from 'dayjs';
  import { LikeFilled, LikeOutlined } from '@ant-design/icons-vue';
  import { BUCKET_OMS_IMAGES } from '/@/enums/assetsEnum';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { h, ref } from 'vue';
  import $api from '../api';
  // import { useUserStore } from '/@/store/modules/user';
  import { inject } from 'vue';

  const getCommentList = inject('getCommentList');
  const { createConfirm } = useMessage();
  // const userStore = useUserStore();
  const props = defineProps({
    comments: {
      type: Array,
      default: () => [],
    },
    guideId: {
      type: String,
    },
    reply: {
      type: Number,
    },
    del: {
      type: Number,
    },
    currentUserId: {
      type: Number,
    },
    repliedUserName: {
      type: String,
    },
  });
  // const userInfo = ref({
  //   uid: userStore.userInfo.id,
  //   userName: userStore.userInfo.userName,
  //   avatar: 'https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif',
  // });
  // const replyInfo = ref({
  //   uid: '',
  //   blogUid: 'uid000003',
  //   replyUserUid: 0,
  //   avatar: 'https://zos.alipayobjects.com/rmsportal/ODTLcjxAfvqbxHnVXCYX.png',
  // });
  const repliedId = ref();
  function replyTo(uid) {
    const currentComment = props.comments.find((item) => item.id == uid);
    currentComment.commnetFlag = !currentComment.commnetFlag;
    console.log(props.comments, '修改test');
    console.log(currentComment, '修改test11111');
    let lists = document.getElementsByClassName('comment');
    for (let i = 0; i < lists.length; i++) {
      lists[i].style.display = 'none';
    }
    document.getElementById(uid).style.display = 'block';
    if (currentComment.commnetFlag) {
      document.getElementById(uid).style.display = 'block';
      console.log(currentComment.commnetFlag, '我是弹框校验');
    } else {
      document.getElementById(uid).style.display = 'none';
    }
    // if (document.getElementById(uid).style.display = 'block') {
    //   currentComment.commnetFlag = false;
    //   console.log(currentComment.commnetFlag, '我是弹框校验');
    // } else {
    //   currentComment.commnetFlag = true;
    // }
    console.log(lists, '我是list');

    console.log(uid, '我是list里面的uid,也就是评论id');
    repliedId.value = uid;
  }
  async function like(commentId) {
    // 找到对应的评论对象
    const comment = props.comments.find((c) => c.id === commentId);
    if (comment) {
      // 切换点赞状态
      // 更新点赞数
      const isDel = comment.isStar == 1 ? 1 : 0;
      comment.isStar = comment.isStar == 1 ? 0 : 1;
      console.log(comment, '我是cpmment');
      const res = await $api.starComment({ commentId, isDel });
      console.log(res, '我是星星');
      getCommentList();
    }
  }
  // 删除评论
  async function deleteComment(commentId) {
    // 找到对应的评论对象
    // const index = props.comments.findIndex((c) => c.uid === commentUid);
    console.log(commentId);
    createConfirm({
      iconType: 'warning',
      title: () => h('span', '提示'),
      content: () => h('span', '确认删除吗?'),
      onOk: async () => {
        const res = await $api.deleteComment({ commentId });
        getCommentList();
      },
    });
  }
  // 获取域名进而获取头像
  const herfLocation = window.location.host.split(':')[0];
</script>
<style scoped>
  .comment {
    display: none;
  }
  :deep .ant-comment-content-author-name {
    display: flex;
    color: black;
  }
  :deep .ant-comment-content-author-name img {
    width: 52px;
    height: 18px;
    margin-left: 10px;
  }
  .activeLike {
    color: #ff4d4f;
  }
  .commentContentColor {
    color: #3d8bf2;
  }
</style>
