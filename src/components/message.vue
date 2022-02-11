<template>
    <div
        class="message"
    >
        <p class="m-name">{{ from }}</p>
        <p class="m-id">@ {{ id }}</p>
        <div class="m-content" v-html="content"></div>
        <p class="m-time">{{ when }}</p>
    </div>
</template>

<script>
export default {
    props: {
        from: String,
        id:String,
        content: String,
        when: String
    }
}
</script>

<style>
@import '../assets/cooldesign.css';
.message {
  border-left: solid rgb(233, 233, 233) 5px;
  padding-left: 10px;
  font-size: 18px;
  margin: 0 0 18px 0;
}
.m-name {
  font-weight: bold;
  margin: 0;
}
.m-id {
  margin: 0;
  color: rgb(122, 122, 122);
  font-size: 12px;
}

.m-content {
  margin: 5px 0 5px 0;
  white-space: pre-line;
  line-height: 1;
}
.m-content h1{
  font-weight: 500;
  font-size: 30px;
  margin: 6px 0;
}
.m-content h2,.m-content h3,.m-content h4,.m-content h5,.m-content h6{
  font-size: 22px;
  margin: 0;
  font-weight: 500;
}
.m-content p{
  margin: 0;
}
.m-content blockquote{
  margin: 0;
  padding-left: 10px;
  border-left: #818fae solid 4px;
}
.m-content pre{
  margin: 0px;
  padding: 10px;
  border-radius: 10px;
  background-color: #21252b;
}
.m-content ol,ul{
  margin: 0;
  line-height: 0.8;
}
.m-content img{
  max-width: 765px;
  max-height: 600px;
  border-radius: 10px;
}

.m-time {
  margin: 0;
  color: rgb(122, 122, 122);
  font-size: 16px;
}

@media screen and (max-width: 840px) {
  .m-content img{
    max-width: 100%;
  }    
}
@media (prefers-color-scheme: dark) {
  .message {
    border-color: #818fae;
  }
  .m-name, .m-content {
    color: white;
  }
}
</style>