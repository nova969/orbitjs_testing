// Orbit.js Guide - v0.16
// Getting Started - Part 2

const server = module.exports = {};
import Orbit, { Schema } from "@orbit/data";

import MemorySource from "@orbit/memory";
import APIStore from "@orbit/jsonapi";
import Sequelize from "sequelize";
import Coordinator, { SyncStrategy, RequestStrategy } from "@orbit/coordinator";

import jagql from "@jagql/framework";
import SQLStore from "@jagql/store-sequelize";
import { Joi } from "jsonapi-server";
import jsonApi from "json-api";

import fetch from 'node-fetch';

Orbit.fetch = fetch;

var schemaDefinition = {
    models: {
        contact: {
            attributes: {
                name: { type: "string" },
                meta: { type: "string" },
                id: { type: "int" }
            }
            /** /,
            relationships: {
                planet: { type: "hasOne", model: "planet", inverse: "moons" }
            }
            /**/
        }
    },
    pluralize: val => val,
    singularize: val => val
};

const schema = new Schema(schemaDefinition);

var RelationalDbStore = require("jsonapi-store-relationaldb");
var sequelize = new Sequelize("jsonapi", "{DB_USER}", "{DBPASSW}", { dialect: "mysql", inverse: "contact" });

const relationalDbStore = new SQLStore({ sequelize });

jagql.setConfig({
    graphiql: true,

    swagger: {
        title: 'Example JSON:API Server',
        version: '0.1.1',
        description: 'This is the API description block that shows up in the swagger.json',
        termsOfService: 'http://example.com/termsOfService',
        contact: {
            name: 'API Contact',
            email: 'apicontact@holidayextras.com',
            url: 'docs.hapi.holidayextras.com'
        },
        license: {
            name: 'MIT',
            url: 'http://opensource.org/licenses/MIT'
        },
        security: [
            {
                'APIKeyHeader': []
            }
        ],
        securityDefinitions: {
            APIKeyHeader: {
                type: 'apiKey',
                in: 'header',
                name: 'X-API-Auth'
            }
        }
    },
    protocol: 'http',
    hostname: 'localhost',
    port: 16006,
    base: 'rest',

    meta: {
        description: 'This block shows up in the root node of every payload XXX'
    }
});

jagql.define({
    resource: "contact",
    attributes: {
        name: Joi.string(),
        meta: Joi.string(),
        type: Joi.string(),
        id: Joi.number()
    },

    handlers: relationalDbStore,

    protocol: 'http',
    hostname: 'localhost',
    port: 16006,
    base: 'rest',

    meta: {
        description: 'This block shows up in the root node of every payload'
    }
});


jagql.authenticate((request, callback) => {
    // If a "blockMe" header is provided, block access.
    if (request.headers.blockme) return callback(new Error('Fail'))

    // If a "blockMe" cookie is provided, block access.
    if (request.cookies.blockMe) return callback(new Error('Fail'))

    return callback()
});


// If we're using the example server for the test suite,
// wait for the tests to call .start();
if (typeof describe === 'undefined') {
    jagql.start(() => {
        console.log('Server running on http://localhost:16006 - X');
    });
}

server.start = jagql.start;
server.close = jagql.close;
server.getExpressServer = jsonApi.getExpressServer;

const memory = new MemorySource({ schema });

const backup = new APIStore({
    schema,
    /**/
    name: "backup",
    namespace: "somenamespace",
/**/
    defaultFetchSettings: {
        protocol: 'http',
        host: 'localhost',
        port: 16006,
        base: 'rest',
        namespace: "contact"
    }
});

const coordinator = new Coordinator({
    sources: [memory, backup]
});

const backupStoreSync = new SyncStrategy({
    source: "memory",
    target: "backup",

    blocking: true
});
/**/
// Update the remote server whenever the store is updated
coordinator.addStrategy(new RequestStrategy({
    source: 'memory',
    on: 'beforeUpdate',

    target: 'backup',
    action: 'update',

    blocking: true
}));

/**/
//coordinator.addStrategy(backupStoreSync);

const bernd = {
    type: "contact",
    id: 1,
    meta: "bern",
    attributes: {
        name: "NodeJS Bernd"
    }
};
const frank = {
    type: "contact",
    id: 2,
    meta: "frn",
    attributes: {
        name: "NodeJS Frank"
    }
};

const io = {
    type: "contact",
    id: 3,
    meta: "THIS IS A IO",
    attributes: {
        meta: "THIS IS A IO ATTRIBUTE",
        name: "IO!!! "
    }
};

(async () => {
    // pull all data from backup and sync it with the store
    let remoteContacts = await backup.pull(q => q.findRecords("contact").sort("name"));
    /**/
    //let transform = await backup.pull(q => q.findRecords());

    console.log("\n\n\nMY CONTACTS: ", JSON.stringify(remoteContacts));

    await memory.sync(remoteContacts);

    let contacts = await memory.query(q => q.findRecords("contact").sort("name"));
    console.log("\n\n\nplanets - loaded from backup", contacts);

    await coordinator.activate().then(() => {
        console.log('Coordinator is active');
    });

/** /
    await memory.update(t => [
        //t.addRecord(frank),
        //t.addRecord(bernd),
        t.addRecord(io) // ADD WORKS - ADDS Entry to my MySQL DB
    ]);
/**/
    contacts = await memory.query(q => q.findRecords("contact").sort("name"));
    console.log("planets in store - after update", contacts);
    await backup.update(t => t.addRecord({
        type: "contact",
        id: 300,
        meta: "THIS IS A TEST",
        attributes: {
            meta: "META ATTRIBUTE",
            name: "THE NAME!!"
        }
    })); // UPDATE/DELETE Does not work...
/**/
})();
