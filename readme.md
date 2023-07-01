<p align="center">
  <a href="https://expressjs.com/" target="blank"><img src="https://miro.medium.com/v2/resize:fit:1400/1*XP-mZOrIqX7OsFInN2ngRQ.png" width="200" alt="Express Logo" /></a>
</p>

# Book Api

## Description of the project

This project is a web application where you can manage authors, books and images of books. The application is developed with Node.js, Express.js, MySQL and Sequelize ORM.

## Characteristics of the project

1. Create, read, update and delete authors.

2. Create, read, update and delete books.

## Technologies used

1. Express:  A web application framework for Node.js. It is designed for building web applications and APIs.

2. Sequelize: Sequelize is a promise-based Node.js ORM for Postgres, MySQL, MariaDB, SQLite and Microsoft SQL Server.

3. MySQL: MySQL is an open-source relational database management system.

4. Express-validator: Express middleware for the validator module.

## Requirements to use the project

After cloning the repository, you must install Node.js. If you don't have it installed, you can download it from the following link: https://nodejs.org/es/

You must also install MySQL. If you don't have it installed, you can download it from the following link: https://www.mysql.com/downloads/

Or you can use Docker for Mysql. If you don't have it installed, you can download it from the following link: https://www.docker.com/products/docker-desktop

## Installation of the project

1. Clone the repository.
2. Install the dependencies with the command: `npm install`.
3. Create a file called `.env` in the root of the project and copy the content of the file `.env.template` in the `.env` file.
4. Add the environment variables in the `.env` file.
5. If you are using Docker, you must run the command: `docker-compose up -d` to start the database.
6. If you are not using Docker, you must create the database manually.
7. Run the command: `npm run dev` to start the project in development mode.