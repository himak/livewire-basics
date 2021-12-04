# Laracasts - Livewire Basics

Tutorial from [laracasts.com](https://laracasts.com/series/livewire-basics)

More informations about [Livewire](https://laravel-livewire.com/)

## Section 1 - Introduction

### Episode 1 - How it Works
    
    // Install Livewire
    composer require livewire/livewire
    
    // Create a livewire component
    php artisan make:livewire counter

    // Add to main layout
    @livewireStyles
    @livewireScripts

    // Insert livewire component to blade view
    <livewire:counter />

    // app/Http/Livewire/Counter.php
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }
    
    // resources/views/livewire/counter.blade.php
    <div>
        <span>{{ $count }}</span>
        <button wire:click="increment">+</button>
    </div>


How Livewire works:

    // vendor/livewire/livewire/src/Controllers/HttpConnectionHandler.php
    
    class HttpConnectionHandler extends ConnectionHandler
    {
        public function __invoke()
        {
            $this->applyPersistentMiddleware();
        
            return $this->handle(
                request([
                    'fingerprint',
                    'serverMemo',
                    'updates',
                ])
            );
        }
    ...
    

    // vendor/livewire/livewire/src/Connection/ConnectionHandler.php

    abstract class ConnectionHandler
    {
        public function handle($payload)
        {
            return LifecycleManager::fromSubsequentRequest($payload)
                ->boot()
                ->hydrate()
                ->renderToView()
                ->dehydrate()
                ->toSubsequentResponse();
        }
    }

    // vendor/livewire/livewire/src/LifecycleManager.php

    public static function fromSubsequentRequest($payload)
    {
        dd( tap(new static, function ($instance) use ($payload) {
            $instance->request = new Request($payload);
            $instance->instance = app('livewire')->getInstance($instance->request->name(), $instance->request->id());
        }) );
    }

next step:

    // vendor/livewire/livewire/src/LifecycleManager.php

    public function hydrate()
    {
        foreach (static::$hydrationMiddleware as $class) {
            $class::hydrate($this->instance, $this->request);
        }
        
        dd($this); // show down

        return $this;
    }
    
you can view in browser:
    
    // dd($this);

    Livewire\LifecycleManager {#370 ▼
        +request: Livewire\Request {#377 ▼
            +fingerprint: array:5 [▼
                "id" => "jf4DaXulYyCxD0MPC9p3"
                "name" => "counter"
                "locale" => "en"
                "path" => "/"
                "method" => "GET"
                ]
            +updates: array:1 [▶]
            +memo: array:5 [▼
                "children" => []
                "errors" => []
                "htmlHash" => "95c683c9"
                "data" => array:1 [▼
                    "count" => 0
                    ]
                "dataMeta" => []
            ]
        }
        +instance: App\Http\Livewire\Counter {#376 ▼
            +count: 1
            +id: "jf4DaXulYyCxD0MPC9p3"

    
    // vendor/livewire/livewire/src/LifecycleManager.php

    public function toSubsequentResponse()
    {
        dd($this);

        return $this->response->toSubsequentResponse();
    }

you can view in browser:

    Livewire\LifecycleManager {#370 ▼
        +request: Livewire\Request {#377 ▶}
        +instance: App\Http\Livewire\Counter {#376 ▶}
        +response: Livewire\Response {#394 ▼
            +request: Livewire\Request {#377 ▶}
            +fingerprint: array:5 [▶]
            +effects: array:2 [▼
                "html" => """
                    <div>
                        Counter component
                        <div>
                            <span>1</span>
                            <button wire:click="decrement">-</button>
                            <button wire:click="increment">+</button>
                        </div>
                    </div>
                    """
                "dirty" => array:1 [▶]
            ]
            +memo: array:6 [▶]
        }
    }
