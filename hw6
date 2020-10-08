from bs4 import BeautifulSoup # pip install bs4
from typing import List
import requests # pip install requests
import urllib
import multiprocessing
import time
import click

start_url = 'https://ru.wikipedia.org/wiki/Теория струн'
end_url = 'https://ru.wikipedia.org/wiki/Математика'

# visited['Some Article'] is a pair:
# (distance: int, parents = [parent1, parent2, ...])
# distance: the length of the path from the start_url
# parents: a list of strings containing articles of the level "distance-1"
#   that link to the current one.
visited = {}
current_level = {}

def linked_articles_from_text(text):
    soup = BeautifulSoup(text, features="html.parser")
    content = soup.find("div", {"id": "mw-content-text"})
    
    links = content.find_all("a")
    
    for link in links:
        href = link.get('href', '')
        
        if href.startswith('/wiki/'): 
            pagename = requests.utils.unquote(href[6:])
            internal_page = False
            for prefix in ['Файл:', 'Служебная:', 'Шаблон:', 'Википедия:', 'Портал:']:
                if pagename.startswith(prefix):
                    internal_page = True
            if not internal_page:
                yield pagename.split('#')[0]
            

def process_page(pagename):
    try:
        article_text = requests.get('https://ru.wikipedia.org/wiki/' + requests.utils.quote(pagename)).text
    except:
        time.sleep(2)
        article_text = requests.get('https://ru.wikipedia.org/wiki/' + requests.utils.quote(pagename)).text
    article_links = list(linked_articles_from_text(article_text))
    # print('Processing article: ' + pagename)
    return((pagename, article_links))


def explore_wiki(start, goal, nthreads):
    global visited, current_level, next_set
    visited = {}
    current_level[start] = (0, [])
    next_set = {}
    distance = 0

    pool = multiprocessing.Pool(nthreads)

    while not current_level.get(goal):
        links = pool.map(process_page, current_level)

        visited.update(current_level)
        current_level = {}
        distance += 1

        for (parent, children) in links:
            for child in set(children):
                if visited.get(child):
                    continue
                if not current_level.get(child):
                    # the article has not been mentioned yet
                    current_level[child] = (distance, [parent])
                else:
                    current_level[child][1].append(parent)
    visited.update(current_level)

# returns one of the shortest paths
def shortest_path(from_article: str, to_article: str, n_threads: int) -> List[str]:
    explore_wiki(from_article, to_article, n_threads)
    path = []
    current = to_article

    while current != from_article:
        path.append(current)
        # visited[current][1] is the list of all parents on the previous level
        current = visited[current][1][0]
    path.reverse()
    return path
    
# this function assumes that the visited global variable is already filled
def get_all_paths(goal):
    # paths: a list of lists, each element is a sequence of articles
    current_paths = []
    next_paths = [ [goal] ]

    reached_source = False
    while not reached_source:
        current_paths = next_paths
        next_paths = []
        for path in current_paths:
            # take the last article in the current chain,
            # iterate over all parents
            article = path[-1]
            for parent in visited[article][1]:
                new_path = path.copy()
                new_path.append(parent)
                next_paths.append(new_path)
            # if there are no parent links to follow, stop
            if not visited[article][1]:
                reached_source = True

    return current_paths

def format_paths(paths):
    text = ''

    for path in paths:
        rev_path = path.copy()
        rev_path.reverse()
        text += ' => '.join(rev_path).replace('_', ' ') + '\n'
    return text[:-1]

@click.command()
@click.argument('from_article')
@click.argument('to_article')
def shortest_paths(from_article, to_article):
    explore_wiki(from_article.replace(' ', '_'), to_article.replace(' ', '_'), 10)
    all_paths = get_all_paths(to_article.replace(' ', '_'))
    print(format_paths(all_paths))

if __name__ == '__main__':  
    shortest_paths()
    # explore_wiki('Теория_струн', 'Скалярная_кривизна', 10)
    # all_paths = get_all_paths('Скалярная_кривизна')
    # print(format_paths(all_paths))
