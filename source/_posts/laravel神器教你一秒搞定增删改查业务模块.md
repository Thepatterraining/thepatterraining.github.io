---
title: laravel神器教你一秒搞定增删改查业务模块
date: 2020-04-08 17:56:47
tags: ['php','laravel']
category: php
article: laravel神器教你一秒搞定增删改查业务模块
---

# laravel神器教你一秒搞定增删改查业务模块

还在为了不断写增删改查而烦恼不堪嘛？还在为了重复写代码而头疼嘛？这个laravel神器拯救你的大脑，解放你的双手。让你有更多的时间去写出更好的代码。

## 安装

首先使用composer安装

> composer require thepatter/query-common

安装之后创建一个command

> php artisan make:command MakeQueryCommand

<!--more-->

把下面的内容复制粘贴进去

```php
<?php

namespace App\Console\Commands;

use Illuminate\Support\Str;
use InvalidArgumentException;
use Illuminate\Console\GeneratorCommand;
use Symfony\Component\Console\Input\InputOption;

class MakeQueryCommand extends GeneratorCommand
{
    /**
     * The console command name.
     *
     * @var string
     */
    protected $name = 'make:queryController';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Create a new queryController class';

    /**
     * The type of class being generated.
     *
     * @var string
     */
    protected $type = 'QueryController';

    /**
     * Get the stub file for the generator.
     *
     * @return string
     */
    protected function getStub()
    {
        return resource_path('stubs/queryController.stub');
    }

    /**
     * Get the default namespace for the class.
     *
     * @param  string  $rootNamespace
     * @return string
     */
    protected function getDefaultNamespace($rootNamespace)
    {
        return $rootNamespace.'\Http\Controllers';
    }

    /**
     * Build the class with the given name.
     *
     * Remove the base controller import if we are already in base namespace.
     *
     * @param  string  $name
     * @return string
     */
    protected function buildClass($name)
    {
        $controllerNamespace = $this->getNamespace($name);

        $replace = [];

        if ($this->option('model')) {
            $replace = $this->buildModelReplacements($replace);
        }

        $replace["use {$controllerNamespace}\Controller;\n"] = '';

        return str_replace(
            array_keys($replace), array_values($replace), parent::buildClass($name)
        );
    }

    /**
     * Build the model replacement values.
     *
     * @param  array  $replace
     * @return array
     */
    protected function buildModelReplacements(array $replace)
    {
        $modelClass = $this->parseModel($this->option('model'));

        if (! class_exists($modelClass)) {
            if ($this->confirm("A {$modelClass} model does not exist. Do you want to generate it?", true)) {
                $this->call('make:model', ['name' => $modelClass]);
            }
        }

        return array_merge($replace, [
            'DummyFullModelClass' => $modelClass,
        ]);
    }

    /**
     * Get the fully-qualified model class name.
     *
     * @param  string  $model
     * @return string
     */
    protected function parseModel($model)
    {
        // if (preg_match('([^A-Za-z0-9_/\\\\])', $model)) {
        //     throw new InvalidArgumentException('Model name contains invalid characters.');
        // }

        $model = trim(str_replace('/', '\\', $model), '\\');

        if (! Str::startsWith($model, $rootNamespace = $this->laravel->getNamespace())) {
            $model = $rootNamespace.$model;
        }

        return $model;
    }

    /**
     * Get the console command options.
     *
     * @return array
     */
    protected function getOptions()
    {
        return [
            ['model', 'm', InputOption::VALUE_OPTIONAL, 'Generate a query controller for the given model.'],
        ];
    }
}

```

在resources文件夹下创建stubs文件夹，在stubs文件夹下面创建QueryController.stub文件，把下面内容复制粘贴进去

```php
<?php

namespace DummyNamespace;

use Illuminate\Http\Request;
use DummyRootNamespaceHttp\Controllers\QueryList\QueryController;
use App\Exceptions\CommonException;

class DummyClass extends QueryController
{
    /**
     * 字典数组
     * ['表里的字段名' => '字典code',...]
     */
    protected $dicArr = [];

    /**
     * 字段映射 可选，不填默认转成下划线格式
     * ['搜索字段' => '表字段',...]
     */
    protected $filedsAdapter = [];

    /**
     * 创建时候的字段映射 可选，不填默认转成下划线格式
     * ['输入字段' => '表字段']
     */
    protected $createAdapter = [];

    //定义表名 格式: table as t
    protected $shortTableName;


    protected function getModel() {
        return new "DummyFullModelClass";
    }

    /*
     * 查询列表
     * @route get.api/lists
     */
    public function getList(Request $request){
        try{
            //检查页码，搜索条件等
            $this->pageValid();
            //返回数据
            return $this->success($this->pageList());
        } catch (Exception $ex) {
            
        }
        
    }  

    /**
     * 创建
     * @route post.api/info
     */
    function createInfo(Request $request) {
        try{
            //创建
            $this->create($request->all());
            return $this->success(true);
        }catch(Exception $ex) {

        }
    }

    /**
     * 更新
     * @route put.api/info/{id}
     */
    function updateInfo(Request $request, $id) {
        try{
            //查询记录
            $detail = $this->getModel()->find($id);
            if (empty($detail)) {
                //补充错误信息
                throw new CommonException();
            }
            //更新
            $this->update($id,$request->all());
            return $this->success(true);
        }catch(Exception $ex) {

        }
    }

    /**
     * 查询一条记录
     * @route get.api/info
     */
    function detail(Request $request) {
        try{
            $rules = [
                'id'=>'required',
            ];
            $messages = [
                'id.required'=>'id为必填项',
            ];
            //验证
            $this->valid($request, $rule, $messages);
            //查询记录
            $detail = $this->getModel()->find($id);
            if (empty($detail)) {
                //补充错误信息
                throw new CommonException();
            }
            return $this->success($detail);
        }catch(Exception $ex) {

        }
    }

    /**
     * 删除一条记录
     * @route delete.api/info/{id}
     */
    function deleteInfo(Request $request, $id) {
        try{
            //查询记录
            $model = $this->getModel();
            $detail = $model->find($id);
            if (empty($detail)) {
                //补充错误信息
                throw new CommonException();
            }
            //进行删除
            $res = $model->where('id', $id)->delete();
            return $this->success(true);
        }catch(Exception $ex) {

        }
    }
        
}

```

## 创建业务逻辑

这时候执行创建的`artisan`命令就可以了

> php artisan make:queryController your controller path -m your model path

这时候在你的Controller下面就会多出一个Controller文件，你只需要在路由中添加路由就可以了。

这个库的github地址在下面，感兴趣的朋友可以看一下。

> https://github.com/Thepatterraining/queryCommon