<!-- 点击标签后跳转的页面-->
<script>
  import PocketBase from "pocketbase";
  import { PocketBase_URL } from "../utils/api/index";
  import { currentchannelid, currentnoticeid, selectedtag } from "../store.js";
  import { push } from "svelte-spa-router";
  import { onMount } from "svelte";

  const pb = new PocketBase(PocketBase_URL);
  let records = [];
  let tags = [];
  async function noticedisplay() {
    try {
      const channel = $currentchannelid;
      const selected = $selectedtag;
      const responses = await pb.collection("notices").getFullList({
        sort: "-created",
        filter: `channelid="${channel}"`,
      });
      const names = responses.map((response) => response.tag);
      tags = [...new Set(names)];
      if (selected) {
        // 确保selected有值
        records = responses.filter((response) => response.tag === selected);
      } else {
        records = responses;
      }
    } catch (error) {
      alert("fail to find notices");
    }
  }
  onMount(() => {
    noticedisplay();
  });

  function send() {
    push("/postnotice");
  }

  function check(id) {
    currentnoticeid.set(id);
    push("/checknotice");
  }

  function origin(tagname) {
    selectedtag.set(tagname);
    noticedisplay();
  }
</script>

<button on:click={send}>发送通知</button>

<div class="container">
  <div class="left">
    {#each records as record}
      <div
        class="record"
        role="button"
        tabindex="0"
        on:click={() => check(record.id)}
        on:keypress
      >
        <a href="#/checknotice" class="title">{record.tittle}</a>
        <div class="author">#{record.tag} from:{record.username}</div>
        <div class="content">
          {record.body.length > 10
            ? `${record.body.substring(0, 10)}...`
            : record.body}
        </div>
      </div>
    {/each}
  </div>
  <div class="right">
    <h2 class="h2">Tags</h2>
    {#each tags as tag}
      <a href="#/jumptag" class="tag" on:click={() => origin(tag)}>{tag}</a>
    {/each}
  </div>
</div>

<style>
  .h2 {
    text-align: left;
    color: #ffffff;
  }
  .record {
    width: 80%;
    padding: 40px;
    background-color: #ffffff;
    text-align: left;
  }
  .record:hover {
    color: #ffffff;
    opacity: 1;
    background-color: #e9ebeb;
  }
  .tag {
    width: 20%;
    padding: 10px;
    /* color: #ffffff; */
  }
  .title {
    text-decoration: underline; /* 为标题添加下划线 */
    color: #0000ee;
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 10px;
  }

  .content {
    font-size: 16px;
    color: black;
  }

  .author {
    font-size: 14px;
    color: #666;
  }
  .container {
    display: flex;
    width: 100%;
    justify-content: space-between; /* 两侧对齐 */
    align-items: flex-start; /* 顶部对齐 */
    padding: 10px;
  }
  .left {
    flex-grow: 1; /* 让两边占据相等的空间，也可以根据需要调整 */
  }
  .right {
    flex-basis: 20%;
  }
</style>
