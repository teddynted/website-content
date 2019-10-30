
```
static async getInitialProps(props) {
        const  { query, store, asPath } = props.ctx
        const slug = query.slug;
        store.dispatch(getPostBySlug(slug));
        await new Promise(resolve => {
            const unsubscribe = store.subscribe(() => {
              unsubscribe();
              resolve();
            });
        });
        const { post_by_slug } = store.getState();
        store.dispatch(getGithubMdByFile(post_by_slug[0].post_body))
        return {
            canonical: apiConfig.baseURL + "/" + asPath,
            post_by_slug
        };
} 
```
