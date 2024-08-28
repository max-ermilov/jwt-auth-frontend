# jwt-auth-frontend
Пример входа по логину и паролю, работы по JWT с обновлением access-токена по refresh-токену из cookies

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Profile</title>
</head>
<body>
  <h1>Profile</h1>
  <form id="profileForm">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" readonly>
    <br>
    <label for="name">Name:</label>
    <input type="text" id="name" name="name">
    <br>
    <button type="submit">Save Changes</button>
  </form>

  <script>
    const BASE_URL = 'https://api.test.com';

    document.addEventListener('DOMContentLoaded', () => {
      const token = localStorage.getItem('accessToken');
      if (!token) {
        window.location.href = '/login.html'; // Перенаправление на страницу входа, если нет токена
        return;
      }

      // Функция для обновления access токена
      const refreshAccessToken = () => {
        return fetch(`${BASE_URL}/refresh-token`, {
          method: 'POST',
          headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json',
          })
          .then(res => {
            if (!res.ok) {
              return Promise.reject('Failed to refresh access token');
            }
            return res.json();
          })
          .then(data => {
            localStorage.setItem('accessToken', data.accessToken);
            return data.accessToken;
          });
      };

      // Функция для выполнения запроса с проверкой токена
      const fetchWithToken = (url, options) => {
        return fetch(url, options).then(response => {
          if (response.status === 401) {
            // Токен истек, пытаемся обновить access токен
            return refreshAccessToken().then(newToken => {
              options.headers['Authorization'] = `Bearer ${newToken}`;
              return fetch(url, options);
            });
          }
          return response;
        });
      };

      // Получение информации о пользователе с учетом обновления токена
      fetchWithToken(`${BASE_URL}/users/me`, {
        method: 'GET',
        headers: {
          'Accept': 'application/json',
          'Authorization': `Bearer ${token}`
        }
      })
      .then(response => {
        if (!response.ok) {
          return Promise.reject('Failed to fetch user data');
        }
        return response.json();
      })
      .then(data => {
        document.getElementById('email').value = data.email;
        document.getElementById('name').value = data.name || '';
      })
      .catch(error => {
        console.error(error);
        alert('Failed to load profile data.');
      });

      // Сохранение изменений профиля
      const profileForm = document.getElementById('profileForm');
      profileForm.addEventListener('submit', (e) => {
        e.preventDefault();
        const name = document.getElementById('name').value;

        fetchWithToken(`${BASE_URL}/users/me`, {
          method: 'PATCH',
          headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`
          },
          body: JSON.stringify({ name })
        })
        .then(response => {
          if (!response.ok) {
            return Promise.reject('Failed to update profile');
          }
          return response.json();
        })
        .then(data => {
          alert('Profile updated successfully!');
        })
        .catch(error => {
          console.error(error);
          alert('Failed to update profile.');
        });
      });
    });
  </script>
</body>
</html>
```
