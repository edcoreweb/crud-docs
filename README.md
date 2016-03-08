# Basic Task List

- [Introduction](#introduction)
- [Installation](#installation)
	- [Installing The Package](#installing-the-package)
	- [Dependencies](#dependencies)
- [Setting Up The Models](#setting-up-the-models)
	- [Eloquent Model](#eloquent-model)
	- [Crud Engine](#crud-engine)
- [Routing](#routing)
	- [The Crud Controller](#the-crud-controller)
- [Concrete Controllers & Views](#concrete-controllers-views)
	- [The Single Page Controller](#the-single-page-controller)
	- [The Single Page View](#the-single-page-view)
- [Handling Editing And Adding](#handling-editing-and-adding)
	- [The Edit View](#the-edit-view)
	- [The Save Action](#the-save-action)

<a name="introduction"></a>
## Introduction

This quickstart guide provides a basic introduction to the crud Concrete5.7 package and includes content on creating and managing CRUD admin tables.

To sample a basic selection of this package features, we will build a simple task list we can use to track all of the tasks we want to accomplish. In other words, the typical "to-do" list example. The complete, finished source code for this project is [available on GitHub](https://github.com/laravel/quickstart-basic).

<a name="installation"></a>
## Installation

<a name="installing-the-package"></a>
#### Installing The Package

Of course, first you will need a installation of the Concrete5.7 cms. All you need to do is to copy the package in the `packages` folder.

<a name="dependencies"></a>
#### Dependencies

 This package relies heavily on the Laravel framework's eloquent models and ORM. Below are the required dependencies.

	"illuminate/database": "4.1.*"
    "illuminate/validation": "4.1.*"
    "hazzard/validation": "^0.1"


<a name="setting-up-the-models"></a>
## Setting Up The Models

<a name="eloquent-model"></a>
### Eloquent Model

First, let's create a simple eloquent model. We will create a model for the `users` table. You should create all your eloquent models in the `src/Models` folder. You can create it in any folder as long as you namespace it correctly.
The `src/` folder is namespaced under `Five`, as such the model namespace will be `Five\Models`.

	<?php
    
    namespace Five\Models;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'users';
    
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
    
    }

If your table's primary key name is not `id`, you should add the following property to your table.

    /**
     * The primary key for the model.
     *
     * @var string
     */
    protected $primaryKey = 'your_primary_key_here';


For more info on Laravel models read the [documentation](https://laravel.com/docs/4.2/eloquent).

<a name="crud-engine"></a>
### Crud Engine

Let's create our crud engine for the model created above. We will call it `UserCrud`. You should create all your crud engines in the `src/Models/Crud` folder. You can create it in any folder as long as you namespace it correctly.
The `src/` folder is namespaced under `Five`, as such the engine namespace will be `Five\Models\Crud`.

	<?php
    
    namespace Five\Models\Crud;
    
    use Five\Crud\Engine;
    
    class UserCrud extends Engine
    {
        /**
         * Set the crud table Eloquent select.
         *
         * @return \Illuminate\Database\Eloquent\Builder;
         */
        public function getModel()
        {
            return \Five\Models\User::select('*');
        }
    }
    
You need to implement the `getModel()` method and return a `Illuminate\Database\Eloquent\Builder` instance. In our case we selected all the columns in the table. This is our start select.

We'll learn more about how to use the crud engine as we add routes to our application.

<a name="routing"></a>
## Routing

<a name="the-crud-controller"></a>
### The Crud Controller

Next, we are ready to create the routing controller. All contact with the crud engine will go through this controller. You should create all your crud engines in the `src/Controllers` folder. You can create it in any folder as long as you namespace it correctly.
The `src/` folder is namespaced under `Five`, as such the engine namespace will be `Five\Controllers`.

It's indicated to respect the folder structure because the namespaces are shorter and readable.

For this application, we know we will need at least three routes: a route to display a list of all of our tasks, a route to add new tasks, and a route to delete existing tasks. We'll wrap all of these routes in the `web` middleware so they have session state and CSRF protection. So, let's stub all of these routes in the `app/Http/routes.php` file:

	<?php
	
    namespace Five\Controllers;
    
    use Five\Crud\Api\Controller;
    
    class UserCrudController extends Controller
    {
        /**
         * The base path on which all routes and urls will be made.
         *
         * @var string
         */
        protected $baseRoute = '/dashboard/users';
    
        /**
         * Set up the crud engine.
         *
         * @return \Five\Crud\Engine $engine
         */
        protected function getEngine()
        {
            return new \Five\Models\Crud\UserCrud;
        }
    }
    
You need to implement the `getEngine()` method and return a `Five\Crud\Engine` instance. In our case we'll return a new instance of the crud engine created above.
You also need to set the `$baseRoute` property to the single page view you want to display the crud in. In our case this is the `dashboard/users` page.

<a name="concrete-controllers-views"></a>
### Concrete Controllers & Views

<a name="the-single-page-controller"></a>
### The Single Page Controller

Next we'll create the single page controller. (In our case is a dashboard single page controller).
The path of the controller will be : `/controllers/single_page/dashboard/users.php`

    <?php
    
    namespace Concrete\Package\Romcrew\Controller\SinglePage\Dashboard;
    
    use \Concrete\Core\Page\Controller\DashboardPageController;
    use Five\Controllers\UserCrudController;
    
    class Users extends DashboardPageController
    {
        /**
         * Override for view function
         */
        public function view()
        {
            $crud = new UserCrudController();
    
            $response = UserCrudController::getView($crud->get());
    
            $this->set('response', $response);
        }
    }

Notice the namespace. I have this namespace because the controller is in a package called `Romcrew`.
If you create the single page in the `application/` folder, your namespace will be `Application\Controller\SinglePage\Dashboard`

So we create a instance of our controller and we get the view response with the `getView($data)` method, then we inject it into concrete's view.


<a name="the-single-page-view"></a>
### The Single Page View

We created the Concrete single page controller, now let's do the view.
We named our controller `Users` and as per concrete's naming conventions the file `users.php`.
Our view file will be called `view.php` and be located in the `single_pages/dashboard/users` folder as one of concrete's naming conventions dictate.

    <?php
    
    View::element('search', $response, 'romcrew');
    
That's it. We pass the response to the `search` element and the crud table will be built.

<a name="handling-editing-and-adding"></a>
## Handling Editing and Adding

This crud only has a single view which lists all the entries. From here on we'll handle the editing and adding.

<a name="the-edit-view"></a>
### The Edit View

We'll create the `single_pages/dashboard/users/edit.php` file.

    <form action="<?= $this->action('save', $model->id) ?>" method="post">
        <div class='row'>
            <div class='col-md-12'>
                <div class="page-header">
                    <h2><?= t('User details') ?></h2>
                </div>
            </div>
        </div>
    
        <div class='row'>
            <div class='col-md-6'>
                <div class="form-group">
                    <label for="username">Username</label>
                    <input type="text" class="form-control" name="username" value="<?= $model->username ?>" placeholder="Username">
                </div>
    
                <div class="form-group">
                    <label for="email">Email</label>
                    <input type="email" class="form-control" name="email" value="<?= $model->email ?>" placeholder="Email address">
                </div>
            </div>
        </div>
    
        <div class="ccm-dashboard-form-actions-wrapper">
            <div class="ccm-dashboard-form-actions">
                <a href="<?= URL::to('/dashboard/users') ?>" class="btn btn-danger pull-left"><?= t('Cancel') ?></a>
                <div class="btn-group pull-right">
                    <input type="submit" class="btn btn-success" name="add" value="<?= t('Save') ?>" />
                    <input type="submit" class="btn btn-default" name="more" value="<?= t('Save and new') ?>" />
                </div>
            </div>
        </div>
    </form>
    
So, a few pointers:
+ the form action `$this->action('save', $model->id)` where `$model->id` is the primary key of your eloquent model.
+ notice that all fields have the same name as the database table counterpart.
+ the submit group - `URL::to('/dashboard/users')` when we press cancel we want to go back to the users page.

<a name="the-save-action"></a>
### The Save Action

Next, we need to define our save action in the concrete single page controller created previously at `/controllers/single_page/dashboard/users.php`.

We'll add the following methods :

    /**
         * Renders the edit/add page.
         *
         * @param int|string|null $id
         */
        public function edit($id = null)
        {
            $model = \Five\Models\User::find($id);
    
            if ($model) {
                $this->set('model', $model);
            }
    
            $this->render('/dashboard/users/edit');
        }
    
        /**
         * Handles updating and adding.
         *
         * @param int|null $id
         */
        public function save($id = null)
        {
            $model = \Five\Models\User::firstOrNew(['id' => $id]);
    
            $model->fill([
                'username' => $this->post('username'),
                'email' => $this->post('email')
            ])->save();
    
            $this->oneMore($model->id);
        }
    
        /**
         * Should or should not create one more entry.
         *
         * @param int|string $primary
         */
        private function oneMore($primary)
        {
            if ($this->post('more')) {
                $this->redirect('/dashboard/users/edit');
            }
    
            $this->redirect('/dashboard/users/edit/' . $primary);
        }
        
The `edit($id)` method renders the edit page, E.g.: when we access `\dashboard\users\edit\1`,
The `save($id)` method handles the adding or updating a user with `id` received as parameter.
The `oneMore($id)` method redirects us to the edited user or to an empty add page depending on which button we pressed. (Save or Save and new).
