---
title: react hooks
date: 2019-10-25 21:56:26
tags: react 
---
vue和react的区别
<!-- more -->

| 比较                      | Rect                                          | Vue                                        | 
| :------                   | :------                                       | :------                                    |
| 如何重新渲染                | `setState( {a:2} )`                          | `this.data.a=2`                            |
| shouldUpdateComponent方法 | 有这个方法                                      | 没有这个概念                                |
| 渲染思路                   | 是基于不可变数据来的 需要用户自己判断是否需要重新渲染 | 是基于可变数据的思路来的 观察数据 自动重新渲染模版 |



今天把react看了下 写的react hooks的 todolist

```js
import React, {useReducer, useState} from 'react'
import produce from 'immer';
import {Button, Input, Card, message, Icon} from 'antd'
import 'antd/dist/antd.css';

function reducer (state, action) {
    switch (action.type) {
        case 'toggle':
            return produce(state, (draftState) => {
                draftState[action.payload].isCompleted
                    = !draftState[action.payload].isCompleted
            })
        case 'add':
            return produce(state, (draftState) => {
                draftState.push({label: action.payload})
            })
        case 'delete':
            return produce(state, (draftState) => {
                draftState.splice(action.payload, 1);
            })
        default:
            return state
    }
}

function Todo ({isCompleted, label, onChange, onDelete}) {
    return (<p style={{display: 'flex', justifyContent: 'space-between'}}>
        <label style={{
            textDecoration: isCompleted && 'line-through'
        }}>
            <input
                type="checkbox"
                checked={isCompleted || false}
                onChange={onChange}
            />
            <span>{label}</span>
        </label>
        <Icon type="delete" style={{cursor: 'pointer'}}
              onClick={onDelete}
        />
    </p>)
}

function ToDoList () {
    const todos = [
        {label: 'Do something'},
        {label: 'Buy dinner'}
    ]

    const [state, dispatch] = useReducer(reducer, todos)
    const [newTodo, setNewTodo] = useState('')
    return (
        <Card bordered={false} style={{width: 400, margin: 'auto', marginTop: 100}}>
            {state.map((todo, i) => (
                <Todo
                    key={i}
                    {...todo}
                    onChange={() => dispatch(
                        {type: 'toggle', payload: i}
                    )}
                    onDelete={() => dispatch(
                        {type: 'delete', payload: i}
                    )}
                />
            ))}
            <Input
                type="text"
                value={newTodo}
                style={{width: 280}}
                onChange={
                    (e) => setNewTodo(e.target.value)
                }
            />
            <Button style={{marginLeft: 10}} type="primary" onClick={() => {
                if (!newTodo) {
                    message.info('please input todo');
                    return
                }
                dispatch({type: 'add', payload: newTodo})
                setNewTodo('')
            }}>
                Add
            </Button>
        </Card>
    )
}

export default ToDoList;

```
