from flask import Flask, jsonify, request
import requests
import json
import os
from collections import OrderedDict

app = Flask(__name__)

USER_AGENT = os.environ.get('FooBarApp/3.0', 'NamaAplikasiAnda/VersiAnda')

def get_discogs_data(release_id):
    headers = {'User-Agent': USER_AGENT}
    api_url = f'https://api.discogs.com/releases/{release_id}'
    try:
        response = requests.get(api_url, headers=headers)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"Terjadi kesalahan saat menghubungi API: {e}")
        return None

@app.route('/discogs/release_info/<int:release_id>', methods=['GET'])
def discogs_release_info(release_id):
    release_data = get_discogs_data(release_id)

    if release_data:
        formatted_data = OrderedDict([
            ("catalog_number", release_data.get('labels', [{}])[0].get('catno') if release_data.get('labels') else None),
            ("artist", release_data.get('artists', [{}])[0].get('name') if release_data.get('artists') else None),
            ("title", release_data.get('title')),
            ("genre", release_data.get('genres', [])),
            ("format", [f.get('name') for f in release_data.get('formats', [])]),
            ("label", [label.get('name') for label in release_data.get('labels', [])]),
            ("released_year", release_data.get('year')),
        ])
        response = jsonify(formatted_data)
        response.headers.add('Access-Control-Allow-Origin', '*')
        # Ganti "*" dengan origin spesifik frontend Anda untuk production
        return response
    else:
        response = jsonify({'error': f'Tidak dapat mengambil data rilis dengan ID {release_id}'}), 404
        response.headers.add('Access-Control-Allow-Origin', '*')
        # Ganti "*" dengan origin spesifik frontend Anda untuk production
        return response

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)