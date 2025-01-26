# 100 Advanced Flask Framework Interview Questions and Answers

## Flask Basics and Configuration

1. **Q: What is Flask and what are its key features?**
   - A: Flask is a lightweight WSGI web application framework. Key features include:
   - Built-in development server and debugger
   - RESTful request dispatching
   - Jinja2 templating
   - Secure cookies support
   - Unicode-based
   - Extensive documentation
   - Unit testing support
   - WSGI 1.0 compliant
   - Extension support

2. **Q: How do you configure Flask applications?**
   ```python
   app = Flask(__name__)
   # Method 1: Using config object
   app.config['DEBUG'] = True
   
   # Method 2: From file
   app.config.from_pyfile('config.py')
   
   # Method 3: From object
   app.config.from_object('config.ProductionConfig')
   
   # Method 4: From environment variables
   app.config.from_envvar('APP_CONFIG')
   ```

3. **Q: Explain Flask's Application Factory pattern.**
   - A: The Application Factory pattern is a way to create multiple instances of your application with different configurations:
   ```python
   def create_app(config_filename):
       app = Flask(__name__)
       app.config.from_pyfile(config_filename)
       
       from yourapplication.views import views
       app.register_blueprint(views)
       
       return app
   ```

4. **Q: What are Flask's context globals?**
   - A: Flask has two contexts:
   1. Application Context (`current_app`, `g`)
   2. Request Context (`request`, `session`)
   ```python
   from flask import current_app, g, request, session
   
   @app.route('/')
   def index():
       g.user = get_current_user()  # Store during request
       return current_app.config['KEY']
   ```

5. **Q: How do you handle configuration for different environments?**
   ```python
   class Config:
       SECRET_KEY = 'base-secret-key'
       
   class DevelopmentConfig(Config):
       DEBUG = True
       SQLALCHEMY_DATABASE_URI = 'sqlite:///dev.db'
       
   class ProductionConfig(Config):
       DEBUG = False
       SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')
   ```

## Request Handling

6. **Q: How do you access request data in Flask?**
   ```python
   from flask import request
   
   @app.route('/submit', methods=['POST'])
   def submit():
       # Form data
       form_data = request.form.get('field')
       
       # Query parameters
       query_param = request.args.get('param')
       
       # JSON data
       json_data = request.get_json()
       
       # Files
       file = request.files['file']
   ```

7. **Q: Explain request hooks in Flask.**
   - A: Flask provides several request hooks:
   ```python
   @app.before_request
   def before_request():
       g.user = get_current_user()
   
   @app.after_request
   def after_request(response):
       response.headers['X-Custom'] = 'Value'
       return response
   
   @app.teardown_request
   def teardown_request(exception):
       db.session.remove()
   ```

8. **Q: How do you handle file uploads in Flask?**
   ```python
   from werkzeug.utils import secure_filename
   
   @app.route('/upload', methods=['POST'])
   def upload_file():
       if 'file' not in request.files:
           return 'No file part'
       file = request.files['file']
       if file.filename == '':
           return 'No selected file'
       if file and allowed_file(file.filename):
           filename = secure_filename(file.filename)
           file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
   ```

9. **Q: How do you implement request validation?**
   ```python
   from flask_validator import ValidateRequest, String, Integer
   
   @app.route('/api/user', methods=['POST'])
   @ValidateRequest(
       json={
           'name': [String(), {'required': True}],
           'age': [Integer(), {'min': 0, 'max': 120}]
       }
   )
   def create_user():
       data = request.get_json()
       return jsonify(data)
   ```

10. **Q: How do you handle CORS in Flask?**
    ```python
    from flask_cors import CORS
    
    # Enable CORS for all routes
    CORS(app)
    
    # Enable CORS for specific route
    @app.route('/api/data')
    @cross_origin()
    def get_data():
        return jsonify({'data': 'value'})
    ```

## Response Objects

11. **Q: What are different types of responses in Flask?**
    ```python
    # String response
    @app.route('/string')
    def string_response():
        return 'Hello World'
    
    # JSON response
    @app.route('/json')
    def json_response():
        return jsonify({'message': 'Hello'})
    
    # Custom response
    @app.route('/custom')
    def custom_response():
        return Response('Custom', mimetype='text/plain')
    ```

12. **Q: How do you set response headers?**
    ```python
    @app.route('/headers')
    def with_headers():
        response = make_response('Hello')
        response.headers['X-Custom'] = 'Value'
        response.set_cookie('user_id', '123')
        return response
    ```

13. **Q: How do you stream responses in Flask?**
    ```python
    @app.route('/stream')
    def stream_response():
        def generate():
            for i in range(100):
                yield f"data: {i}\n\n"
                time.sleep(0.1)
        
        return Response(generate(), mimetype='text/event-stream')
    ```

14. **Q: How do you handle file downloads?**
    ```python
    from flask import send_file, send_from_directory
    
    @app.route('/download')
    def download_file():
        return send_file(
            'path/to/file.pdf',
            as_attachment=True,
            download_name='report.pdf'
        )
    ```

15. **Q: How do you implement response compression?**
    ```python
    from flask_compress import Compress
    
    compress = Compress()
    compress.init_app(app)
    
    @app.route('/compressed')
    def compressed_response():
        return jsonify(large_data)
    ```

## Routing and URL Building

16. **Q: How do you implement dynamic routes?**
    ```python
    @app.route('/user/<username>')
    def user_profile(username):
        return f'Profile of {username}'
    
    @app.route('/post/<int:post_id>')
    def show_post(post_id):
        return f'Post {post_id}'
    ```

17. **Q: Explain URL converters in Flask.**
    ```python
    # Custom URL converter
    class ListConverter(BaseConverter):
        def to_python(self, value):
            return value.split(',')
        
        def to_url(self, values):
            return ','.join(values)
    
    app.url_map.converters['list'] = ListConverter
    
    @app.route('/items/<list:item_ids>')
    def show_items(item_ids):
        return f'Items: {item_ids}'
    ```

18. **Q: How do you handle subdomain routing?**
    ```python
    app = Flask(__name__)
    app.config['SERVER_NAME'] = 'example.com:5000'
    
    @app.route('/', subdomain='<subdomain>')
    def subdomain_route(subdomain):
        return f'Subdomain: {subdomain}'
    ```

19. **Q: How do you build URLs dynamically?**
    ```python
    from flask import url_for
    
    @app.route('/')
    def index():
        # Generate URL for user profile
        profile_url = url_for('user_profile', username='john')
        # Generate external URL
        external_url = url_for('index', _external=True)
        return f'URLs: {profile_url}, {external_url}'
    ```

20. **Q: How do you implement route middleware?**
    ```python
    from functools import wraps
    
    def route_middleware(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            # Pre-route logic
            result = f(*args, **kwargs)
            # Post-route logic
            return result
        return decorated_function
    
    @app.route('/')
    @route_middleware
    def protected_route():
        return 'Protected'
    ```

## Templates and Jinja2

21. **Q: How do you use template inheritance in Jinja2?**
    ```html
    <!-- base.html -->
    <!DOCTYPE html>
    <html>
    <head>
        <title>{% block title %}{% endblock %}</title>
    </head>
    <body>
        {% block content %}{% endblock %}
    </body>
    </html>
    
    <!-- child.html -->
    {% extends "base.html" %}
    {% block title %}Page Title{% endblock %}
    {% block content %}
        <h1>Content</h1>
    {% endblock %}
    ```

22. **Q: How do you create custom Jinja2 filters?**
    ```python
    @app.template_filter('reverse')
    def reverse_filter(s):
        return s[::-1]
    
    # In template
    {{ "Hello"|reverse }}
    ```

23. **Q: How do you handle template contexts?**
    ```python
    @app.context_processor
    def utility_processor():
        def format_price(amount):
            return f"${amount:.2f}"
        return dict(format_price=format_price)
    
    # In template
    {{ format_price(100) }}
    ```

24. **Q: How do you implement macros in Jinja2?**
    ```html
    {% macro input(name, value='', type='text') %}
        <input type="{{ type }}" name="{{ name }}"
               value="{{ value|e }}">
    {% endmacro %}
    
    <!-- Usage -->
    {{ input('username') }}
    {{ input('password', type='password') }}
    ```

25. **Q: How do you cache templates?**
    ```python
    from flask_caching import Cache
    
    cache = Cache(app, config={'CACHE_TYPE': 'simple'})
    
    @app.route('/')
    @cache.cached(timeout=300)
    def cached_view():
        return render_template('expensive.html')
    ```

## Database Integration

26. **Q: How do you integrate SQLAlchemy with Flask?**
    ```python
    from flask_sqlalchemy import SQLAlchemy
    
    app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://user:pass@localhost/db'
    db = SQLAlchemy(app)
    
    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(80), unique=True)
    ```

27. **Q: How do you handle database migrations?**
    ```python
    from flask_migrate import Migrate
    
    migrate = Migrate(app, db)
    
    # Command line:
    # flask db init
    # flask db migrate -m "Initial migration"
    # flask db upgrade
    ```

28. **Q: How do you implement database connection pooling?**
    ```python
    from sqlalchemy import create_engine
    
    engine = create_engine('postgresql://user:pass@localhost/db',
                          pool_size=5,
                          max_overflow=10,
                          pool_timeout=30)
    ```

29. **Q: How do you handle database transactions?**
    ```python
    @app.route('/transfer')
    def transfer_money():
        try:
            with db.session.begin_nested():
                # Perform transaction
                account1.balance -= amount
                account2.balance += amount
            db.session.commit()
        except:
            db.session.rollback()
            raise
    ```

30. **Q: How do you implement database sharding?**
    ```python
    class ShardedDatabase:
        def __init__(self, shard_count):
            self.shards = [
                create_engine(f'postgresql://user:pass@shard{i}/db')
                for i in range(shard_count)
            ]
        
        def get_shard(self, key):
            shard_id = hash(key) % len(self.shards)
            return self.shards[shard_id]
    ```

## Flask Extensions

31. **Q: How do you implement user authentication with Flask-Login?**
    ```python
    from flask_login import LoginManager, UserMixin
    
    login_manager = LoginManager()
    login_manager.init_app(app)
    
    class User(UserMixin, db.Model):
        id = db.Column(db.Integer, primary_key=True)
    
    @login_manager.user_loader
    def load_user(user_id):
        return User.query.get(int(user_id))
    ```

32. **Q: How do you implement form handling with Flask-WTF?**
    ```python
    from flask_wtf import FlaskForm
    from wtforms import StringField, PasswordField
    
    class LoginForm(FlaskForm):
        username = StringField('Username')
        password = PasswordField('Password')
    
    @app.route('/login', methods=['GET', 'POST'])
    def login():
        form = LoginForm()
        if form.validate_on_submit():
            return 'Logged in'
        return render_template('login.html', form=form)
    ```

33. **Q: How do you implement RESTful APIs with Flask-RESTful?**
    ```python
    from flask_restful import Resource, Api
    
    api = Api(app)
    
    class UserResource(Resource):
        def get(self, user_id):
            return {'user': user_id}
        
        def put(self, user_id):
            return {'status': 'updated'}
    
    api.add_resource(UserResource, '/user/<int:user_id>')
    ```

34. **Q: How do you implement caching with Flask-Caching?**
    ```python
    from flask_caching import Cache
    
    cache = Cache(app, config={'CACHE_TYPE': 'redis'})
    
    @app.route('/expensive-operation')
    @cache.cached(timeout=300)
    def expensive_operation():
        # Expensive computation
        return result
    ```

35. **Q: How do you implement task queues with Celery?**
    ```python
    from flask_celery import Celery
    
    celery = Celery(app)
    
    @celery.task
    def long_running_task():
        # Time-consuming operation
        pass
    
    @app.route('/task')
    def start_task():
        task = long_running_task.delay()
        return {'task_id': task.id}
    ```

## Security

36. **Q: How do you implement CSRF protection?**
    ```python
    from flask_wtf.csrf import CSRFProtect
    
    csrf = CSRFProtect(app)
    
    @app.route('/protected', methods=['POST'])
    @csrf.exempt
    def unprotected_route():
        return 'Not CSRF protected'
    ```

37. **Q: How do you implement rate limiting?**
    ```python
    from flask_limiter import Limiter
    
    limiter = Limiter(app)
    
    @app.route('/api')
    @limiter.limit("100/day")
    def rate_limited():
        return 'Rate limited endpoint'
    ```

38. **Q: How do you implement secure password hashing?**
    ```python
    from werkzeug.security import generate_password_hash, check_password_hash
    
    class User(db.Model):
        password_hash = db.Column(db.String(128))
        
        def set_password(self, password):
            self.password_hash = generate_password_hash(password)
            
        def check_password(self, password):
            return check_password_hash(self.password_hash, password)
    ```

39. **Q: How do you implement JWT authentication?**
    ```python
    from flask_jwt_extended import JWTManager, create_access_token
    
    jwt = JWTManager(app)
    
    @app.route('/login')
    def login():
        access_token = create_access_token(identity=user.id)
        return {'access_token': access_token}
    ```

40. **Q: How do you implement role-based access control?**
    ```python
    from functools import wraps
    
    def role_required(role):
        def decorator(f):
            @wraps(f)
            def decorated_function(*args, **kwargs):
                if not current_user.has_role(role):
                    abort(403)
                return f(*args, **kwargs)
            return decorated_function
        return decorator
    ```

## Testing

41. **Q: How do you write unit tests for Flask applications?**
    ```python
    import unittest
    
    class TestApp(unittest.TestCase):
        def setUp(self):
            self.app = create_app('testing')
            self.client = self.app.test_client()
            
        def test_home_page(self):
            response = self.client.get('/')
            self.assertEqual(response.status_code, 200)
    ```

42. **Q: How do you mock database calls in tests?**
    ```python
    from unittest.mock import patch
    
    @patch('app.models.User.query')
    def test_user_view(self, mock_query):
        mock_query.get.return_value = User(id=1, name='Test')
        response = self.client.get('/user/1')
        self.assertEqual(response.status_code, 200)
    ```

43. **Q: How do you test Flask-Login protected routes?**
    ```python
    def test_protected_route(self):
        with self.client:
            self.client.post('/login', data={
                'username': 'test',
                'password': 'test'
            })
            response = self.client.get('/protected')
            self.assertEqual(response.status_code, 200)
    ```

44. **Q: How do you implement API testing?**
    ```python
    def test_api_endpoint(self):
        response = self.client.post('/api/resource',
                                  json={'key': 'value'},
                                  headers={'Authorization': f'Bearer {token}'})
        self.assertEqual(response.status_code, 201)
        self.assertIn('id', response.json)
    ```

45. **Q: How do you measure test coverage?**
    ```python
    # Using pytest-cov
    # pytest --cov=app tests/
    
    def test_coverage_example(self):
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'Welcome', response.data)
    ```

## Deployment

46. **Q: How do you deploy Flask applications with Gunicorn?**
    ```python
    # gunicorn.conf.py
    workers = 4
    bind = '0.0.0.0:8000'
    worker_class = 'gevent'
    
    # Command:
    # gunicorn -c gunicorn.conf.py wsgi:app
    ```

47. **Q: How do you implement Docker containerization?**
    ```dockerfile
    FROM python:3.9
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install -r requirements.txt
    COPY . .
    CMD ["gunicorn", "wsgi:app"]
    ```

48. **Q: How do you implement nginx reverse proxy?**
    ```nginx
    server {
        listen 80;
        server_name example.com;
        
        location / {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
    ```

49. **Q: How do you handle static file serving in production?**
    ```python
    # Using WhiteNoise
    from whitenoise import WhiteNoise
    
    app.wsgi_app = WhiteNoise(app.wsgi_app,
                             root='static/',
                             prefix='static/')
    ```

50. **Q: How do you implement SSL/TLS?**
    ```python
    from flask_talisman import Talisman
    
    Talisman(app,
             force_https=True,
             content_security_policy={
                 'default-src': "'self'"
             })
    ```

## Performance and Scaling

51. **Q: How do you implement application profiling?**
    ```python
    from flask_profiler import Profiler
    
    app.config['PROFILE'] = True
    profiler = Profiler(app)
    
    @app.route('/endpoint')
    @profiler.profile()
    def profiled_endpoint():
        return 'Profiled'
    ```

52. **Q: How do you implement request queuing?**
    ```python
    from flask import g
    import queue
    
    request_queue = queue.Queue(maxsize=100)
    
    @app.before_request
    def queue_request():
        try:
            request_queue.put_nowait(request)
        except queue.Full:
            abort(503)
    ```

53. **Q: How do you implement connection pooling?**
    ```python
    from sqlalchemy import create_engine
    from sqlalchemy.pool import QueuePool
    
    engine = create_engine('postgresql:///',
                          poolclass=QueuePool,
                          pool_size=20,
                          max_overflow=0)
    ```

54. **Q: How do you implement load balancing?**
    ```python
    # Using nginx load balancer
    upstream flask_apps {
        server 127.0.0.1:8001;
        server 127.0.0.1:8002;
        server 127.0.0.1:8003;
    }
    ```

55. **Q: How do you implement response compression?**
    ```python
    from flask_compress import Compress
    
    compress = Compress()
    compress.init_app(app)
    
    @app.route('/large-response')
    def compressed_response():
        return large_data
    ```

## Error Handling

56. **Q: How do you implement custom error handlers?**
    ```python
    @app.errorhandler(404)
    def not_found_error(error):
        return render_template('404.html'), 404
    
    @app.errorhandler(500)
    def internal_error(error):
        db.session.rollback()
        return render_template('500.html'), 500
    ```

57. **Q: How do you implement error logging?**
    ```python
    import logging
    from logging.handlers import RotatingFileHandler
    
    handler = RotatingFileHandler('app.log', maxBytes=10000, backupCount=3)
    handler.setLevel(logging.INFO)
    app.logger.addHandler(handler)
    ```

58. **Q: How do you implement API error responses?**
    ```python
    class APIError(Exception):
        def __init__(self, message, status_code=400):
            self.message = message
            self.status_code = status_code
    
    @app.errorhandler(APIError)
    def handle_api_error(error):
        response = jsonify({'error': error.message})
        response.status_code = error.status_code
        return response
    ```

59. **Q: How do you implement validation error handling?**
    ```python
    from marshmallow import Schema, fields, ValidationError
    
    class UserSchema(Schema):
        email = fields.Email(required=True)
    
    @app.route('/api/user', methods=['POST'])
    def create_user():
        try:
            data = UserSchema().load(request.json)
        except ValidationError as err:
            return {'errors': err.messages}, 422
    ```

60. **Q: How do you implement debug mode safely?**
    ```python
    app.config.update(
        DEBUG=True,
        DEBUG_TB_ENABLED=True,
        DEBUG_TB_INTERCEPT_REDIRECTS=False,
        DEBUG_TB_PROFILER_ENABLED=True
    )
    ```

## RESTful APIs

61. **Q: How do you implement API versioning?**
    ```python
    from flask import Blueprint
    
    api_v1 = Blueprint('api_v1', __name__, url_prefix='/api/v1')
    api_v2 = Blueprint('api_v2', __name__, url_prefix='/api/v2')
    
    @api_v1.route('/resource')
    def resource_v1():
        return {'version': 1}
    
    @api_v2.route('/resource')
    def resource_v2():
        return {'version': 2}
    ```

62. **Q: How do you implement API pagination?**
    ```python
    @app.route('/api/items')
    def get_items():
        page = request.args.get('page', 1, type=int)
        per_page = request.args.get('per_page', 10, type=int)
        
        items = Item.query.paginate(page=page, per_page=per_page)
        return {
            'items': [item.to_dict() for item in items.items],
            'total': items.total,
            'pages': items.pages
        }
    ```

63. **Q: How do you implement API rate limiting?**
    ```python
    from flask_limiter import Limiter
    from flask_limiter.util import get_remote_address
    
    limiter = Limiter(
        app,
        key_func=get_remote_address,
        default_limits=["200 per day", "50 per hour"]
    )
    
    @app.route("/api/resource")
    @limiter.limit("1 per second")
    def rate_limited_resource():
        return {"status": "success"}
    ```

64. **Q: How do you implement API authentication?**
    ```python
    from functools import wraps
    from flask import request, g
    
    def require_api_key(f):
        @wraps(f)
        def decorated(*args, **kwargs):
            api_key = request.headers.get('X-API-Key')
            if not api_key or not validate_api_key(api_key):
                return {'error': 'Invalid API key'}, 401
            return f(*args, **kwargs)
        return decorated
    ```

65. **Q: How do you implement API documentation?**
    ```python
    from flask_restx import Api, Resource, fields
    
    api = Api(app, version='1.0', title='Sample API',
              description='A sample API')
    
    user_model = api.model('User', {
        'id': fields.Integer,
        'name': fields.String,
        'email': fields.String
    })
    
    @api.route('/users/<int:id>')
    class User(Resource):
        @api.doc('get_user')
        @api.marshal_with(user_model)
        def get(self, id):
            return get_user(id)
    ```

## Authentication and Authorization

66. **Q: How do you implement OAuth2?**
    ```python
    from authlib.integrations.flask_client import OAuth
    
    oauth = OAuth(app)
    google = oauth.register(
        name='google',
        client_id='your-client-id',
        client_secret='your-client-secret',
        access_token_url='https://accounts.google.com/o/oauth2/token',
        access_token_params=None,
        authorize_url='https://accounts.google.com/o/oauth2/auth',
        authorize_params=None,
        api_base_url='https://www.googleapis.com/oauth2/v1/',
        client_kwargs={'scope': 'openid email profile'}
    )
    ```

67. **Q: How do you implement role-based access control?**
    ```python
    from functools import wraps
    
    def requires_roles(*roles):
        def wrapper(f):
            @wraps(f)
            def wrapped(*args, **kwargs):
                if not current_user.has_roles(roles):
                    abort(403)
                return f(*args, **kwargs)
            return wrapped
        return wrapper
    
    @app.route('/admin')
    @requires_roles('admin')
    def admin_page():
        return 'Admin only!'
    ```

68. **Q: How do you implement session management?**
    ```python
    from flask_session import Session
    
    app.config['SESSION_TYPE'] = 'redis'
    Session(app)
    
    @app.route('/set')
    def set_value():
        session['key'] = 'value'
        return 'Value set'
    
    @app.route('/get')
    def get_value():
        return session.get('key', 'not set')
    ```

69. **Q: How do you implement password reset?**
    ```python
    from itsdangerous import URLSafeTimedSerializer
    
    def generate_reset_token(email):
        serializer = URLSafeTimedSerializer(app.config['SECRET_KEY'])
        return serializer.dumps(email, salt='password-reset-salt')
    
    def verify_reset_token(token, expiration=3600):
        serializer = URLSafeTimedSerializer(app.config['SECRET_KEY'])
        try:
            email = serializer.loads(
                token,
                salt='password-reset-salt',
                max_age=expiration
            )
            return email
        except:
            return None
    ```

70. **Q: How do you implement two-factor authentication?**
    ```python
    import pyotp
    
    def generate_totp_secret():
        return pyotp.random_base32()
    
    def verify_totp(secret, token):
        totp = pyotp.TOTP(secret)
        return totp.verify(token)
    
    @app.route('/2fa/verify', methods=['POST'])
    def verify_2fa():
        token = request.form.get('token')
        if verify_totp(current_user.totp_secret, token):
            session['2fa_verified'] = True
            return redirect(url_for('index'))
        return 'Invalid token', 400
    ```

## Session Management

71. **Q: How do you implement secure session handling?**
    ```python
    from flask_session import Session
    from redis import Redis
    
    app.config.update(
        SESSION_TYPE='redis',
        SESSION_REDIS=Redis(host='localhost', port=6379),
        PERMANENT_SESSION_LIFETIME=timedelta(minutes=30),
        SESSION_COOKIE_SECURE=True,
        SESSION_COOKIE_HTTPONLY=True,
        SESSION_COOKIE_SAMESITE='Lax'
    )
