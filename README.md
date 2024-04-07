[![PHP Unit](https://github.com/Saritasa/php-laravel-chat-api/workflows/PHP%20Unit/badge.svg)](https://github.com/Saritasa/php-laravel-chat-api/actions)
[![PHP CodeSniffer](https://github.com/Saritasa/php-laravel-chat-api/workflows/PHP%20Codesniffer/badge.svg)](https://github.com/Saritasa/php-laravel-chat-api/actions)
[![codecov](https://codecov.io/gh/Saritasa/php-laravel-chat-api/branch/master/graph/badge.svg)](https://codecov.io/gh/Saritasa/php-laravel-chat-api)

# Laravel Chat Api    
Adds chat functionality for your project on top of Laravel [Broadcasting](https://laravel.com/docs/broadcasting) feature.

## Laravel 5.5+
Lower versions are not supported.

### Installation and configuration Install the ```saritasa/laravel-chat-api``` package:    

```bash $ composer require saritasa/laravel-chat-api ```
 Publish config with    
```bash $ artisan vendor:publish --tag=laravel_chat_api ```
 Update config/laravel_chat_api.php sections:    
- Implement IChatUser contract to your application user model and update parameter `userModelClass`.     
- Check notifications section and add your own notification instead this mocks.     

Configure at least one [broadcasting driver](https://laravel.com/docs/broadcasting#driver-prerequisites)

## Work with service Add IChatService contract injection in needed class.    
### Methods: - Create chat    
```php    
 $chatService->createChat($creator, ['name' => 'New Chat'], [1, 2]);    
 ```  
Where [1, 2] - identifiers of participants of chat excluded creator.    

- Close chat    
```php    
 $chatService->closeChat($creator, $chat);  
 ```
 Remember that only creator can close chat and can't close "already closed" chat. In this cases ChatException will be    
thrown.    
- Leave chat    
```php    
 $chatService->leaveChat($user, $chat);  
 ``` 
 When creator leaves chat on of participants became to creator.  
- Send message in chat    
```php    
 $chatService->sendMessage($sender, $chat, $message);  
 ```
 - Mark chat as read    
```php    
 $chatService->markChatAsRead($chat, $user);  
 ```
 ### Events:  
## Chat events:  
Throws in chat channel for all subscribers who participate in chatting.  

- MessageSentEvent throws when participant sent new message in chat.  
- ChatLeavedEvent throws when one of participants leaved this chat.  
- ChatClosedEvent throws when creator closed this chat.  
## User events:  
Throws in user channel for all subscribers who participate in chatting exclude event initiator.  
- ChatCreatedEvent throws when user created new chat.  
- MessageSentEvent throws when participant sent new message in chat.  
- ChatClosedUserEvent throws when creator closed this chat.  
- ChatReopenedUserEvent throws when creator reopened closed chat. 
## Broadcasting settings:
To integrate it with Laravel broadcasting you can just update your broadcasting routes file ( routes/channels.php by default ) with next example:
```php
...
use Saritasa\LaravelChatApi\Events\ChatCreatedEvent;  
use Saritasa\LaravelChatApi\Events\ChatEvent;
use Saritasa\LaravelChatApi\Contracts\IChatUser;
...
// Using model binding for chat events channel.
Broadcast::channel(ChatEvent::CHANNEL_PREFIX . '{chat}', function ($user, Chat $chat) {  
  // Checking that auth user is chat participant.
  return $chat->getUsers()->pluck('id')->contains($user->id);  
});  

Broadcast::channel(ChatCreatedEvent::CHANNEL_PREFIX . '{id}', function ($user, int $id) {  
  return (int)$user->id === $id;  
});
```
Read in official [documentation](https://laravel.com/docs/broadcasting#defining-authorization-callbacks) about defining authorization callbacks.
**Now all chat events will dispatch through your broadcast driver.** 
## Notifications settings:
To easy sent notifications you should create your own [notification](https://laravel.com/docs/notifications#creating-notifications) and update `laravel_chat_api.php` config file.
**Here is example if we should send notification when chat was reopened:**
```php
return [
	...
	'notifications' => [  
	   ... 
	  'chatReopened' => App\Notifications\MyOwnChatReopenedNotification::class,
	],
];
```

## Contributing    
1. Create fork    
2. Checkout fork    
3. Develop locally as usual. **Code must follow [PSR-1](http://www.php-fig.org/psr/psr-1/), [PSR-2](http://www.php-fig.org/psr/psr-2/)**  	
4. Update README.md to describe new or changed functionality. Add changes description to CHANGE file.    
5. When ready, create pull request    

## Resources    
 * [Bug Tracker](http://github.com/saritasa/php-laravel-chat-api/issues)    
* [Code](http://github.com/saritasa/php-laravel-chat-api)
