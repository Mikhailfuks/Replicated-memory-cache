const redis = require('redis');

// Настройка Redis-клиентов
const client1 = redis.createClient({ url: 'redis://localhost:6379' });
const client2 = redis.createClient({ url: 'redis://localhost:6380' });

// Обещания подключения
const client1Promise = client1.connect();
const client2Promise = client2.connect();

// Функция получения данных из кэша
async function getFromCache(key) {
 try {
  await client1Promise; // Ожидаем подключения клиента 1
  await client2Promise; // Ожидаем подключения клиента 2

  // Получаем значение из кэша с помощью обоих клиентов
  const value1 = await client1.get(key);
  const value2 = await client2.get(key);

  // Возвращаем значение, если оно присутствует в одном из кэшей
  if (value1) return value1;
  if (value2) return value2;

  return null; // Значение не найдено
 } catch (error) {
  console.error('Ошибка при получении данных из кэша:', error);
  return null;
 }
}

// Функция записи данных в кэш
async function setToCache(key, value) {
 try {
  await client1Promise; // Ожидаем подключения клиента 1
  await client2Promise; // Ожидаем подключения клиента 2

  // Записываем значение в оба кэша
  await client1.set(key, value);
  await client2.set(key, value);
 } catch (error) {
  console.error('Ошибка при записи данных в кэш:', error);
 }
}

// Пример использования
async function main() {
 const key = 'myKey';
 const value = 'myValue';

 // Запись в кэш
 await setToCache(key, value);
 console.log('Данные записаны в кэш.');

 // Чтение из кэша
 const cachedValue = await getFromCache(key);
 console.log('Значение из кэша:', cachedValue);

 // Отключение клиентов
 client1.quit();
 client2.quit();
}

main();
