---
title: React评论展示案例(包含知识点：state、props、ref、React声明周期、localStorage本地存储等)
subtitle: react
date: 2018-3-29
categories: 案例
tags: 
    - React
---
本案例在上一篇的案例(React组件之间通过Props传值的技巧(小案例，帮助体会理解props、state、受控组件和非受控组件等))的基础上加强功能和用户体验，但是当然还有很多需要改进的地方，后期一步步慢慢增强：    
```javascript
import React,{Component} from 'react';
import {render} from 'react-dom';
import './index.css';

class CommentInput extends Component{
    constructor(){
        super();
        this.state={
            username:'',
            content:''
        }
    }

    handleUsernameChange=(event)=>{
        this.setState({
            username:event.target.value
        })
    };

    handleContentChange=(event)=>{
        this.setState({
            content:event.target.value
        })
    };

    handleSubmit=()=>{
        if(this.props.submit){
            this.props.submit({
                username:this.state.username,
                content:this.state.content,
                createTime:+new Date()
            })
        }
        this.setState({
            content:''
        })
    };

    handleUsernameHold=(event)=>{
        localStorage.setItem('username',event.target.value)
    };

    componentWillMount(){
        const username=localStorage.getItem('username');
        if(username){
            this.setState({username})
        }
    }

    componentDidMount(){
        this.input.focus();
    };

    render(){
        return(
            <div className='comment-input'>
                <div className='comment-field'>
                    <span className='comment-field-name'>用户名：</span>
                    <div className='comment-field-input'>
                        <input
                            ref={(input)=>this.input=input}
                            value={this.state.username}
                            onBlur={this.handleUsernameHold}
                            onChange={this.handleUsernameChange}
                        />
                    </div>
                </div>
                <div className='comment-field'>
                    <span className='comment-field-name'>评论内容：</span>
                    <div className='comment-field-input'>
                        <textarea
                            value={this.state.content}
                            onChange={this.handleContentChange}
                        />
                    </div>
                </div>
                <div className='comment-field-button'>
                    <button onClick={this.handleSubmit}>
                        发布
                    </button>
                </div>
            </div>
        )
    }
}

class CommentList extends Component{

    constructor(){
        super();
        this.state={
            items:[]
        }
    }

    render(){
        return(
            <div>
                {this.props.items.map((item,index)=><Comment deleteItem={this.props.deleteItem} item={item} index={index} key={index}/>)}
            </div>
        )
    }
}

class Comment extends Component{
    constructor(){
        super();
        this.state={
            timeString:''
        }
    }

    handleTimeString=()=>{
        const item=this.props.item;
        const duration=(+Date.now()-item.createTime)/1000;
        return duration>60?`${Math.round(duration/60)}分钟前`:`${Math.round(Math.max(duration,1))}秒前`;
    };

    handleDelete=()=>{
        if(this.props.deleteItem){
            this.props.deleteItem(this.props.index)
        }
    };

    render(){
        return(
            <div className='comment'>
                <div className='comment-user'>
                    <span className='comment-username'>{this.props.item.username} </span>：
                </div>
                <p>{this.props.item.content}</p>
                <span className="comment-delete" onClick={this.handleDelete}>删除</span>
                <span className="comment-createdtime">
                    {this.handleTimeString()}
                </span>
            </div>
        )
    }
}

class CommentApp extends Component{
    constructor(){
        super();
        this.state={
            items:[]
        }
    }

    handleSubmit=(item)=>{
        this.state.items.push(item);
        this.setState({
            items:this.state.items
        });
        localStorage.setItem('items',JSON.stringify(this.state.items))
    };

    handleDelete=(index)=>{
        console.log(index);
        this.state.items.splice(index,1);
        this.setState({
            items:this.state.items
        });
        localStorage.setItem('items',JSON.stringify(this.state.items))
    };

    componentWillMount(){
        let items=localStorage.getItem('items');
        if(items){
            items=JSON.parse(items);
            this.setState({items})
        }
    };

    render(){
        return(
            <div className="wrapper">
                <CommentInput submit={this.handleSubmit} />
                <CommentList deleteItem={this.handleDelete} items={this.state.items}/>
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
  position: relative;
  display: flex;
  border-bottom: 1px solid #f1f1f1;
  margin-bottom: 10px;
  padding-bottom: 10px;
  min-height: 50px;
}

.comment .comment-user {
  flex-shrink: 0;
}

.comment-username {
  color: #00a3cf;
  font-style: italic;
}

.comment-createdtime {
  padding-right: 5px;
  position: absolute;
  bottom: 0;
  right: 0;
  padding: 5px;
  font-size: 12px;
}

.comment:hover .comment-delete {
  color: #00a3cf;
}

.comment-delete {
  position: absolute;
  right: 0;
  top: 0;
  color: transparent;
  font-size: 12px;
  cursor: pointer;
}

.comment p {
  margin: 0;
  /*text-indent: 2em;*/
}

code {
  border: 1px solid #ccc;
  background: #f9f9f9;
  padding: 0px 2px;
}
```

