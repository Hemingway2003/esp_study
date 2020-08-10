Due to the proxy problem, I added a proxy in urllib.
I found my system use python2 as it default python environment, of cource it could change to python3 but use the default python2 is possible.
Here is how I re-edit the `idf-tools.py`:
1. I need a proxy software and get the ip:port just like `127.0.0.1:7890`
2. Added the `urllib` version check code, due to the python2 use `urllib2` while python3 use `urllib.request`
```
URL_LIB_2 = 0
try:
    from urllib.request import urlretrieve
except ImportError:
    from urllib import urlretrieve
    URL_LIB_2 = 1

try:
    from urllib.request import ProxyHandler
except ImportError:
    from urllib2 import ProxyHandler

try:
    from urllib.request import build_opener
except ImportError:
    from urllib2 import build_opener

try:
    from urllib.request import install_opener
except ImportError:
    from urllib2 import install_opener
    import urllib2
```
3. Change the download way:
```
def download(self, version):
        assert(version in self.versions)
        download_obj = self.versions[version].get_download_for_platform(self._platform)
        if not download_obj:
            fatal('No packages for tool {} platform {}!'.format(self.name, self._platform))
            raise DownloadError()

        url = download_obj.url
        # Proxy
        proxy = ProxyHandler({'http': '127.0.0.1:7890'})
        opener = build_opener(proxy)
        install_opener(opener)

        archive_name = os.path.basename(url)
        local_path = os.path.join(global_idf_tools_path, 'dist', archive_name)
        mkdir_p(os.path.dirname(local_path))

        if os.path.isfile(local_path):
            if not self.check_download_file(download_obj, local_path):
                warn('removing downloaded file {0} and downloading again'.format(archive_name))
                os.unlink(local_path)
            else:
                info('file {0} is already downloaded'.format(archive_name))
                return

        downloaded = False
        for retry in range(DOWNLOAD_RETRY_COUNT):
            local_temp_path = local_path + '.tmp'
            info('Downloading {} to {}'.format(archive_name, local_temp_path))
            try:
                if URL_LIB_2 == 1:
                    with open(local_temp_path, 'wb') as f:
                        f.write(urllib2.urlopen(url).read())
                        f.close()
                else:
                    urlretrieve(url, local_temp_path, report_progress if not global_non_interactive else None)
                sys.stdout.write("\rDone\n")
                
                # warn('Running...')
            except Exception as e:
                # urlretrieve could throw different exceptions, e.g. IOError when the server is down
                # Errors are ignored because the downloaded file is checked a couple of lines later.
                warn('Download failure {}'.format(e))
            sys.stdout.flush()
            if not os.path.isfile(local_temp_path) or not self.check_download_file(download_obj, local_temp_path):
                warn('Failed to download {} to {}'.format(url, local_temp_path))
                continue
            rename_with_retry(local_temp_path, local_path)
            downloaded = True
            break
        if not downloaded:
            fatal('Failed to download, and retry count has expired')
            raise DownloadError()
```
