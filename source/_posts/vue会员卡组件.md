---
title: vue实现一个会员卡的组件(可以动态传入图片（分出的一个组件）、背景、文字、卡号等)
subtitle: vue
date: 2018-4-7
categories: 案例
tags: 
    - VUE
---
自己在写这个组件的时候主要遇到的问题就是在动态传入背景图片或者背景色的时候没能立马顺利写出来，不过现在实现了这个简单组件就和大家分享一下
```javascript
<template>
    <div class="card" :style="bg != undefined ? setBg() : {backgroundColor: '#fff'}">
        <div class="cardTop">
            <span class="left">
                <span>
                    <avatar class="leftImg" :src="cardImg" alt=""></avatar>
                </span>
                <span class="content">
                    <span class="cardName">{{cardName != undefined ? cardName : defaultCardName}}</span>
                    <span class="cardType">{{cardType != undefined ? cardType : defaultCardType}}</span>
                </span>
            </span>
            <span class="right">
                <avatar :src="QRCode" alt=""></avatar>
            </span>
        </div>
        <div class="cardBottom">
            <span class="cardNum">卡号：{{cardNum != undefined ? cardNum : defaultCardNum}}</span>
        </div>
    </div>
</template>
<script>
    import Avatar from '@/components/avatar/Avatar.vue';
    export default {
        components: {
            Avatar,
        },
        props: {
            cardImg: {},
            QRCode: {},
            bg: {},
            cardName: {
                type: String,
            },
            cardType: {
                type: String,
            },
            cardNum: {
                type: String,
            },
        },
        data() {
            return {
                backGround: '#fff',
                defaultCardName: '阿里云',
                defaultCardType: '会员卡',
                defaultCardNum: '8888 8888 8888'
            }
        },
        computed: {},
        methods: {
            setBg() {
                let cur = this.bg.lastIndexOf('.');
                
                let img = this.bg.substr(cur + 1);
                
                if (/(gif|jpg|jpeg|png|GIF|JPG|PNG)$/.test(img)) {
                    return { backgroundImage: 'url(' + this.bg + ')' }
                } else {
                    return { backgroundColor: this.bg }
                }
            }
        }
    }
</script>
<style scoped lang="less">
    
    .card {
        height: 180px;
        max-width: 350px;
        display: flex;
        flex-direction: column;
        border-radius: 10px;
        padding: 10px;
        justify-content: space-between;
        border: 1px solid #ccc;
        .cardTop {
            display: flex;
            justify-content: space-between;
            .left {
                display: flex;
                flex-direction: row;
                .leftImg {
                    height: 70px;
                    width: 70px;
                }
                .content {
                    margin-left: 10px;
                    min-height: 70px;
                    display: flex;
                    flex-direction: column;
                    justify-content: space-around;
                    .cardName, .cardType {
                        font-size: 18px;
                    }
                }
            }
            .right {
                img {
                    height: 50px;
                    width: 50px;
                }
            }
        }
        .cardBottom {
        
        }
    }

</style>
```
```css
<template>

    <span :class="avatarCls">
        <img :src="src" v-if="src">
        <i v-else-if="icon" :class="['iconfont', `icon-${icon}`]"></i>
        <span v-else :class="`${prefixCls}-string`" :style="style" ref="children">
            <slot></slot>
        </span>
    </span>
</template>
<script>
export default {
  name: "Avatar",
  data() {
    return {
      prefixCls: "ei-avatar",
      scale: 1,
      isSlotShow: false,
      style: {}
    };
  },
  props: {
    size: {
      type: String,
      default: "large"
    },

    src: String,
    shape: {
      type: String,
      default: "square"
    },
    icon: String
  },
  computed: {
    avatarCls() {
      const size = { large: "lg", small: "sm" }[this.size];

      return [
        this.prefixCls,
        `${this.prefixCls}-${this.shape}`,
        {
          [`${this.prefixCls}-${size}`]: !!size,
          [`${this.prefixCls}-icon`]: !!this.icon,
          [`${this.prefixCls}-image`]: !!this.src
        }
      ];
    },
  },
  methods: {
    setScale() {
      this.isSlotShow = !this.src && !this.icon;
      if (this.$refs.children) {
        const childrenWidth = this.$refs.children.offsetWidth;
        const avatarWidth = this.$el.getBoundingClientRect().width;

        if (avatarWidth - 8 < childrenWidth) {
          this.scale = (avatarWidth - 8) / childrenWidth;
        } else {
          this.scale = 1;
        }
      }
    }
  },
  mounted() {
    this.setScale();
  },
  updated() {
    this.setScale();
  }
};
</script>
<style scoped lang="stylus">

.ei-avatar {
  display: inline-block;
  flex-center(start, center, center)
  text-align: center;
  background: $avatar-bg;
  color: $avatar-color;
  white-space: nowrap;
  position: relative;
  overflow: hidden;
  avatar-size($avatar-size-base, $avatar-font-size-base);

  &-lg {
    avatar-size($avatar-size-lg, $avatar-font-size-lg);
  }

  &-sm {
    avatar-size($avatar-size-sm, $avatar-font-size-sm);
  }

  &-square {
    border-radius: $avatar-border-radius;
  }

  & > img {
    width: 100%;
    height: 100%;
    display: block;
  }
}
</style>
```

