---
title: React组件之间通过Props传值的技巧(小案例，帮助体会理解props、state、受控组件和非受控组件等)
subtitle: props
date: 2018-3-14
categories: 案例
tags: 
    - React
---
本文重要是根据react小书上的一个很简单的例子改编的，加上自己的学习理解，希望可以通过实际案例让大家对概念有更清晰的理解，当然也希望能一块学习。
```javascript
import React,{Component} from 'react';
import {render} from 'react-dom';
import './comprehensive.css'

/*
 * 一个重要问题知识点：
 *可以看到这个CommentInput组件和CommentList组件都是两个子组件，但是开始是两个毫不相干的子组件，我们想让他们建立连接，但是同为两个子级组件之间是没法直接传值的，这时候他们的父级组件CommentApp就起到了一个桥梁的作用
 * ->当用户点击发布按钮的时候我们将CommentInput中的state当中最新的评论数据传递给父级组件CommentApp，然后从父级组件CommentApp传给子组件CommentList进行渲染
 *
 * 问题来了：那么子级CommentInput怎么向父级CommentApp传值呢？
 *   父组件CommentApp只需要通过props给子组件CommentInput传入一个回调函数，当用户点击发布按钮的时候CommentInput调用props中的回调函数并将state传入该函数即可
 */

//这个是commentInput子组件，负责用户输入的区域
class CommentInput extends Component{
    //这里初始化一个state值来保存用户名(username)值和评论内容(content)
    //但是自行设置一个初始化的参数之后就不能从界面中输入了，因为输入控件被设置成了固定值，永远以设置的为准。
    constructor(){
        super();
        this.state={
            username:'',
            content:''
        }
    }

    //为了解决被控问题，所以就要setState一下，监听输入框中的onChange事件，获取到用户输入的内容之后用setState更新一下状态，这样input里边的内容才会发生改变。下同
    handleUsernameChange=(event)=>{
        this.setState({
            //这里我们通过event.target.value获取到输入框中的内容，并通过setState把它设置到state.username中，这样内容就会更新，输入就再没问题了。下同
            username:event.target.value
        })
    };

    handleContentChange=(event)=>{
        this.setState({
            content:event.target.value
        })
    };

    handleSubmit=()=>{
        //这个方法判断props中是否传入了onSubmit属性，有的话就用该函数，并把用户输入的数据都传给该函数
        if(this.props.onSubmit){
            const {username,content}=this.state;
            this.props.onSubmit({
                username,content
            })
        }
        //增加用户体验，提交后清空原来的值
        this.setState({
            username:'',
            content:''
        })
    };

    render(){
        return(
            <div className="comment-input">
                <div className="comment-field">
                    <span className="comment-field-name">用户名：</span>
                    <div className="comment-field-input">
                        //上边是设置state值初始化，这里取值就用this.state.XXX,下边的评论内容同理
                        //这里加上onChange事件就是为了监听input的值是否发生改变，从而更新状态
                        <input type="text" value={this.state.username}
                               onChange={this.handleUsernameChange}
                        />
                    </div>
                </div>
                <div className="comment-field">
                    <span className="comment-field-name">评论内容</span>
                    <div className="comment-field-input">
                        <textarea value={this.state.content}
                                  onChange={this.handleContentChange}
                        />
                    </div>
                </div>
                <div className="comment-field-button">
                    //给按钮添加点击事件
                    <button onClick={this.handleSubmit}>发布</button>
                </div>
            </div>
        )
    }
}

//评论列表子组件
class CommentList extends Component{
    //这里给组件加上defaultProps防止comments不传而报错的情况
    static defaultProps={
        comments:[]
    };

    render(){
        return(
            <div>
                //遍历数组将值传递给Comment
                {this.props.comments.map((comment,index)=>{
                    return <Comment comment={comment} key={index}/>
                })}
            </div>
        )
    }
}

//评论列表中的每个单独项，这个组件被commentList所用
class Comment extends Component{
    render(){
        return(
            //接收到CommentList传递过来的每一条数据然后将其渲染在页面上
            <div className="comment">
                <div className="comment-user">
                    <span className="comment-username">{this.props.comment.username}</span>
                </div>
                <p>{this.props.comment.content}</p>
            </div>
        )
    }
}


//评论功能整体用这个组件包起来，包括commentInput区域和commentList区域
class CommentApp extends Component{
    //在这里初始化一个数组，来保存所有的评论数据，并且通过props把它传递给CommentList
    constructor(){
        super();
        this.state={
            comments:[]
        }
    }

    handleSubmitComment=(comment)=>{
        //当用户发布评论的时候就把数据插入到comment中，然后通过setState将数据更新到页面上
        this.state.comments.push(comment);
        this.setState({
            comments:this.state.comments
        })
    };

    render(){
        return(
            <div className="wrapper">
                //内部引入需要的子组件
                //在这个父组件里给CommentInput子组件传入一个onSubmit的属性，这个属性是CommentApp的一个方法，这样CommentInput就可以调用this.props.onSubmit(...)将数据传递给CommentApp
                <CommentInput onSubmit={this.handleSubmitComment}/>
                <CommentList comments={this.state.comments}/>
            </div>
        )
    }
}

class Index extends Component{
    render(){
        return(
            <div>
                <CommentApp/>
            </div>
        )
    }
}

render(<Index/>,document.getElementById('root'));
```
```css
body {
    margin: 0;
    padding: 0;
    font-family: sans-serif;
    background-color: #fbfbfb;
}
.wrapper {
    width: 500px;
    margin: 10px auto;
    font-size: 14px;
    background-color: #fff;
    border: 1px solid #f1f1f1;
    padding: 20px;
}
/* 评论框样式 */
.comment-input {
    background-color: #fff;
    border: 1px solid #f1f1f1;
    padding: 20px;
    margin-bottom: 10px;
}
.comment-field {
    margin-bottom: 15px;
    display: flex;
}
.comment-field .comment-field-name {
    display: flex;
    flex-basis: 100px;
    font-size: 14px;
}
.comment-field .comment-field-input {
    display: flex;
    flex: 1;
}
.comment-field-input input,
.comment-field-input textarea {
    border: 1px solid #e6e6e6;
    border-radius: 3px;
    padding: 5px;
    outline: none;
    font-size: 14px;
    resize: none;
    flex: 1;
}
.comment-field-input textarea {
    height: 100px;
}
.comment-field-button {
    display: flex;
    justify-content: flex-end;
}
.comment-field-button button {
    padding: 5px 10px;
    width: 80px;
    border: none;
    border-radius: 3px;
    background-color: #00a3cf;
    color: #fff;
    outline: none;
    cursor: pointer;
}
.comment-field-button button:active {
    background: #13c1f1;
}
/* 评论列表样式 */
.comment-list {
    background-color: #fff;
    border: 1px solid #f1f1f1;
    padding: 20px;
}
/* 评论组件样式 */
.comment {
    display: flex;
    border-bottom: 1px solid #f1f1f1;
    margin-bottom: 10px;
    padding-bottom: 10px;
    min-height: 50px;
}
.comment .comment-user {
    flex-shrink: 0;
}
.comment span {
    color: #00a3cf;
    font-style: italic;
}
.comment p {
    margin: 0;
    /*text-indent: 2em;*/
}
```
