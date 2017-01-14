# laracasts_testing_laravel

[Link](https://laravel.com/docs/master/testing) para documentação para testes no laravel

## Intro to Application Testing

Laravel possui helpers que podem testar inclusive '$this->visit()' mesmo sem ter nenhum servidor web rodando.

## Beginning Unit Testing

Testes unitários são os testes mais proximos de uma unica ação.
Sugere, criar um pasta unit para os testes unitários. Em que se é possível testar apenas essa pasta específica.
Sugere se tambem criar um arquivo para os testes referido a uma determinada classe, como: 'ProductTest.php'

Exemplo de teste unitario:

```php
<?php

use App\Product;

class ProductTest extends PHPUnit_Framewort_TestCase
{
    function test_a_product_has_name()
    {
        $product = new Product('Fallout 4',59);
        
        $this->assertEquals('Fallout 4', $product->name());
    }
    
    function test_a_product_has_a_cost()
    {
        $product = new Product('Fallout 4',59);
        
        $this->assertEquals(59, $product->cost());
    }    
```
Com as seguintes observações

- Para testes unitários pode se extender apenas PHPUnit_Framewort_TestCase
- Sugere usar conversao de nome dos testes como underline
- Criar o teste antes de implementar a funcionalidade
- Deve ser iniciar os metodos testes com 'test' ou entao utilizar a anotacao /** @test */ antes do metodo


Pode se extrair a parte que repete para um metodo setUp, que o próprio phpunit vai executar antes de cada teste:

```php
<?php

use App\Product;

class ProductTest extends PHPUnit_Framewort_TestCase
{

    protected $product;
    
    public function setUp(){
        $this->product = new Product('Fallout 4',59);
    }
    
    function test_a_product_has_name()
    {
        $this->assertEquals('Fallout 4', $this->product->name());
    }
    
    function test_a_product_has_a_cost()
    {
        $this->assertEquals(59, $this->product->cost());
    }    
```


## More Unit Testing Review

Lembrar que um teste unitario consiste nos passos


- Set
- Action
- Assert

## Testing Eloquent

Os testes com eloquent são testes integrados. Em que criamos dados e depois verificamos esses dados criados.
Sendo assim, criar na pasta integration, com um arquivo para a model que voce vai testar.

Exemplo:

```php
<?php

use App\Article;
use Illuminate\Foundation\Testing\DatabaseTransactions;

class ProductTest extends PHPUnit_Framewort_TestCase
{

    use DatabaseTransactions

    protected $product;
    /** @test */
    function it_fecthes_trending_articles()
    {
        factory(Article::class, 2)->create();
        factory(Article::class, 2)->create(['reads' => 10]);
        $mostPopular = factory(Article::class, 2)->create(['reads' => 20]);
        
        $articles = Article::trending();
        
        $this->asertEquals($mostPopular->id,$articles->first()->id);
    }
   
```
Em que

- Se extende TestCase, pois ele deve possuir metodos do laravel (para acesso ao eloquent por exemplo)
- Deve se usar a trait DatabaseTransaction, para que em todas as transações de banco, seja feito o rollback

## Testing Database Connection

Pode se criar uma database exclusiva para o teste, criando inclusive migrations especficas para o teste.

Em que se pode criar uma nova conexao no arquivo de database do Laravel. (utilizando sqlite por exemplo).

No arquivo phpunit.xml, podemos alterar a variavel de ambiente DB_CONNECTION para utilizar a conexao que foi criada no momento.

## Testing Collaborators

Em uma model criada, pode se testar coisas simples como uma atribuicao de nome ou outros colunas(sem acessar diretamente o banco)

```php
    /** @test */
    public function a_team_has_a_name()
    {
        $team = new Team(['name' => 'Acme']);
        $this->assertEquals('Acme', $team->name);
    }
```
Este teste contempla a validacao do funcionamento do mass assingment

Para verificar se uma exception é disparada nos testes, utiliza se: $this->setExpectedException('Exception');
Sendo que abaixo desse trecho até o fim do teste, deve ser disparada uma exceção

```php
    /** @test */
    public function a_team_has_a_maximum_size()
    {
        $team = factory(Team::class)->create(['size' => 2]);
        $userOne = factory(User::class)->create();
        $userTwo = factory(User::class)->create();
        $team->add($userOne);
        $team->add($userTwo);
        $this->assertEquals(2, $team->count());
        $this->setExpectedException('Exception');
        $userThree = factory(User::class)->create();
        $team->add($userThree);
    }
```

## Homework Solutions

## Regression Testing

Podem existir bugs(em produção), mesmo quando está se usando TDD. Quando isso ocorrer. Usa se o teste de regressão.

Em que usando TDD, cria-se o caso de teste para o erro ainda nao acertado.
Depois do teste criado, acerta-se o problema.

## "Liking" a Model With TDD

Neste teste é demostrando um recurso do phpunit no laravel em que é possivel se comportar como um user logado na base. Utilizando $this->actingAs($user);

É possivel também verificar algum registro no banco utilizando $this->seeInDatabase();

Exemplo do teste utilizando:

```php
/**@test  */
public function a_user_can_like_a_post()
{
    $post = factory(App\Post::class)-create();
    $user = factory(App\User::class)-create();
    
    $this->actingAs($user);
    
    $post-like();
    
    $this->seeInDatabase('likes',[
        'user_id' => $user->id,
        'likeable_id' => $post->id,
        'likeable_type' => get_class($post)
    ]); // Verifica no banco
    
    $this->assertTrue($post->isLiked()); // Verifica via metodo criado na model
}
```

Nesse teste poderia se extrair a criacao de post e do user para um metodo separado. Utilizando user e post como atributos da classe.

Este teste, utiliza relacionamento polimorficos. Ele possui uma tabela de likes, em que chave estrangeira pode ser a chave de varias tabelas, e existe uma coluna que informa qual classe é essa.

Para definir o relacionamento no eloquent(em classe de Post):
```php
public function likes()
{
    return $this->morphMany(Like::class, 'likeable');
}

public function like(){ // funcao que utiliza a relacionamento.
    $like = new Like(['user_id' => auth::id()]);
    
    $this->likes()->save($like);
}
```

## Test Method Refactoring 

È possivel refatorar o teste criado anteriormente, considerando principalmente que varios testes serão criados na mesma classe de testes, utilizando os mesmos codigos referentes parte de set do código.


Antes de tudo, pode se criar um metodo signIn na classe TestCase (que é extendida nos testes do php unit)
Em que é verificado se passamos um user como paramentro, caso nao cria um para depois logar com ele


```php
public function signIn($user = null)
{
    if(! $user ){
        $user = factory(App\User::class)->create();
    }
    
    $this->user = $user;
    
    $this->actingAs($this->user);
}
```

Podemos tambem criar na nossa classe de Post, um metodo de setUp que  cria o post.

Lembrando que para o Phpunit o metodo setUp executa antes de todo metodo

```php
public function setUp(
{
    parernt::setUp();
    
    $this->post = factory(App\Post:class)->create();
    
    $this->signIn();
}
```

Deixando os metodos mais limpos:


```php
/**@test  */
public function a_user_can_like_a_post()
{
    $post-like();
    
    $this->seeInDatabase('likes',[
        'user_id' => $user->id,
        'likeable_id' => $post->id,
        'likeable_type' => get_class($post)
    ]); // Verifica no banco
    
    $this->assertTrue($post->isLiked()); // Verifica via metodo criado na model
}
```


É possivel criar helpers com metodos para teste como 'createPost'.
Exemplo: Criar arquivo test/helpers/functions.

```php
<?php

funcion createPost($atributes = [])
{
    return factory(App\Post::class)->create($atrributes);
}
```


Assim podemos chamar o metodo com ou sem array de parametros desejados.

Basa incluir no autoload do composer em autoload-dev o arquivo:

```json
autoload-dev: {
    classmap:[
        tests/TestCase.php
    ],
    files: [
        tests/helpers/function.php
    ]
},
```
Rodar composer dump-autoload


