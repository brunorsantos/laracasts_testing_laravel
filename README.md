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

