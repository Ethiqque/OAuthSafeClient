import time
import threading
import requests
from requests.auth import HTTPBasicAuth
from requests.exceptions import HTTPError

class OAuthSafeClient:
    def __init__(self, client_id, client_secret, token_url, api_url, rate_limit, time_unit='seconds'):
        self.client_id = client_id
        self.client_secret = client_secret
        self.token_url = token_url
        self.api_url = api_url
        self.rate_limit = rate_limit
        self.time_unit = time_unit
        self.access_token = None
        self.token_expiry = 0
        self.lock = threading.Semaphore(rate_limit)
        self.last_request_time = 0
        
        self.time_multiplier = {'seconds': 1, 'minutes': 60, 'hours': 3600}.get(time_unit, 1)
        self.refresh_token()

    def refresh_token(self):
        try:
            response = requests.post(
                self.token_url,
                auth=HTTPBasicAuth(self.client_id, self.client_secret),
                data={'grant_type': 'client_credentials'}
            )
            response.raise_for_status()
            token_data = response.json()
            self.access_token = token_data['access_token']
            self.token_expiry = time.time() + token_data['expires_in'] - 60
        except HTTPError as http_err:
            print(f'HTTP error occurred: {http_err}')
        except Exception as err:
            print(f'Other error occurred: {err}')

    def make_request(self, endpoint, method='GET', **kwargs):
        with self.lock:
            current_time = time.time()
            
            if current_time >= self.token_expiry:
                self.refresh_token()

            elapsed_time = current_time - self.last_request_time
            if elapsed_time < self.time_multiplier:
                time.sleep(self.time_multiplier - elapsed_time)

            headers = kwargs.get('headers', {})
            headers['Authorization'] = f'Bearer {self.access_token}'
            kwargs['headers'] = headers
            
            try:
                if method.upper() == 'GET':
                    response = requests.get(f'{self.api_url}{endpoint}', **kwargs)
                elif method.upper() == 'POST':
                    response = requests.post(f'{self.api_url}{endpoint}', **kwargs)
                elif method.upper() == 'PUT':
                    response = requests.put(f'{self.api_url}{endpoint}', **kwargs)
                elif method.upper() == 'DELETE':
                    response = requests.delete(f'{self.api_url}{endpoint}', **kwargs)
                else:
                    raise ValueError('Unsupported HTTP method')
                
                response.raise_for_status()
                self.last_request_time = time.time()
                return response.json()
            
            except HTTPError as http_err:
                print(f'HTTP error occurred: {http_err}')
            except Exception as err:
                print(f'Other error occurred: {err}')

    def shutdown(self):
        print("OAuthSafeClient shutdown complete.")

if __name__ == "__main__":
    client = OAuthSafeClient(
        client_id='your_client_id',
        client_secret='your_client_secret',
        token_url='https://example.com/oauth2/token',
        api_url='https://example.com/api',
        rate_limit=5,
        time_unit='seconds'
    )
    
    try:
        response = client.make_request('/some/endpoint')
        print(response)
    finally:
        client.shutdown()
