# Supertest JS

```java
import request from 'supertest';
import assert from 'assert';

describe('Random Dog Image', function() {

    it('responds with expected JSON structure', async function() {
        const response = await request('https://jsonplaceholder.typicode.com')
            .get('/users')
            .expect(200)

        console.log(response.body);

    });


    it('responds with expected JSON structure2',  async function() {

        const response = await request('https://jsonplaceholder.typicode.com/users')
            .get('/users')
            //.get('/users')
            // .expect(200)

        if (response.status !== 200) {
            console.log('errorrrdurrr')
            console.log(response.error);
            console.log('errorrrdurrr2')
        }

        // console.log(response.body);

    });



    it('should POST', async () => {

        const requestBody = {
            userId: 1,
            id: 1,
            title: "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
            body: "quia et suscipit"
        }

        const response = await request('https://jsonplaceholder.typicode.com')
            .post('/posts')
            .send(requestBody)
            .expect(201)
            .then(response => {
                console.log(response.body)
            })
    });


    it('should PUT', async () => {

        const requestBody = {
            userId: 1,
            id: 1,
            title: "SUNTA aut facere repellat provident occaecati excepturi optio reprehenderit",
            body: "QUIA et suscipit"
        }

        const response = await request('https://jsonplaceholder.typicode.com')
            .put('/posts/1')
            .send(requestBody)
            .expect(200)
            .expect(function(res) {
                assert(res.body.title.includes('SUNTA aut facere repellat provident occaecati excepturi optio reprehenderit'));
                assert(res.body.body.includes('QUIA et suscipit'));
            })
            .then(response => {
                console.log(response.body)
            })
    });

});
```


<https://www.testim.io/blog/supertest-how-to-test-apis-like-a-pro/>




