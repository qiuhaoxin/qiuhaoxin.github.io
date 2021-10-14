##

Vue 2.6

用 v-html 指令出来 html 片段，在 IE 浏览器遇到兼容性问题
现象：像是处理文本一样，横向内容显示不完整，但有滚动条

经调试样式发现，只要改动某处节点的样式就会回复正常。

原因：
未明

处理：
利用 hack，在内容绑定到 dom 后，利用 Vue.$nextTick 来设置一个定时器，来改变 dom 的 height 样式。

```Javascript
          this.$nextTick(function(){
            setTimeout(function(){
                let grantTextDom=document.querySelector('.grant-text');
                let height=grantTextDom.style.height;
                if(height=='100%'){
                  grantTextDom.style.height='';
                }else{
                  grantTextDom.style.height='100%'
                }
            },150)
          })

```
