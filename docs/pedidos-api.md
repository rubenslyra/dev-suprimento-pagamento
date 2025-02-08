```
composer create-project --prefer-dist laravel/laravel pedidos-api

cd pedidos-api

php artisan make:model Pedido -m
php artisan make:controller PedidoController --api
php artisan make:request PedidoRequest
php artisan make:resource PedidoResource
php artisan make:service PedidoService
php artisan make:repository PedidoRepository

// Configuração do Docker (docker-compose.yml)
version: '3.8'
services:
  app:
    build: .
    container_name: laravel_app
    restart: unless-stopped
    working_dir: /var/www
    volumes:
      - .:/var/www
    networks:
      - laravel_net

  nginx:
    image: nginx:alpine
    container_name: laravel_nginx
    restart: unless-stopped
    ports:
      - "8000:80"
    volumes:
      - .:/var/www
      - ./docker/nginx:/etc/nginx/conf.d
    networks:
      - laravel_net

  db:
    image: mysql:8.0
    container_name: laravel_db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: pedidos_db
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    networks:
      - laravel_net

networks:
  laravel_net:

// Model (app/Models/Pedido.php)
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Pedido extends Model
{
    use HasFactory;

    protected $fillable = ['cliente', 'produto', 'quantidade', 'status'];
}

// Migration (database/migrations/xxxx_xx_xx_create_pedidos_table.php)
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up()
    {
        Schema::create('pedidos', function (Blueprint $table) {
            $table->id();
            $table->string('cliente');
            $table->string('produto');
            $table->integer('quantidade');
            $table->string('status');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('pedidos');
    }
};

// Repository (app/Repositories/PedidoRepository.php)
namespace App\Repositories;

use App\Models\Pedido;

class PedidoRepository
{
    public function all()
    {
        return Pedido::all();
    }

    public function find($id)
    {
        return Pedido::findOrFail($id);
    }

    public function create(array $data)
    {
        return Pedido::create($data);
    }

    public function update($id, array $data)
    {
        $pedido = Pedido::findOrFail($id);
        $pedido->update($data);
        return $pedido;
    }

    public function delete($id)
    {
        return Pedido::destroy($id);
    }
}

// Service (app/Services/PedidoService.php)
namespace App\Services;

use App\Repositories\PedidoRepository;

class PedidoService
{
    protected $repository;

    public function __construct(PedidoRepository $repository)
    {
        $this->repository = $repository;
    }

    public function listar()
    {
        return $this->repository->all();
    }

    public function buscar($id)
    {
        return $this->repository->find($id);
    }

    public function criar(array $data)
    {
        return $this->repository->create($data);
    }

    public function atualizar($id, array $data)
    {
        return $this->repository->update($id, $data);
    }

    public function excluir($id)
    {
        return $this->repository->delete($id);
    }
}

// Controller (app/Http/Controllers/PedidoController.php)
namespace App\Http\Controllers;

use App\Http\Requests\PedidoRequest;
use App\Services\PedidoService;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class PedidoController extends Controller
{
    protected $service;

    public function __construct(PedidoService $service)
    {
        $this->service = $service;
    }

    public function index()
    {
        return response()->json($this->service->listar());
    }

    public function show($id)
    {
        return response()->json($this->service->buscar($id));
    }

    public function store(PedidoRequest $request)
    {
        return response()->json($this->service->criar($request->validated()), 201);
    }

    public function update(PedidoRequest $request, $id)
    {
        return response()->json($this->service->atualizar($id, $request->validated()));
    }

    public function destroy($id)
    {
        return response()->json($this->service->excluir($id), 204);
    }
}
```
